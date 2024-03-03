# GHAS for platform engineerrs

## Overview

- [ ] [Exercise 7 - Customizing CodeQL Configuration](#exercise-7---customizing-codeql-configuration-15-mins)
- [ ] [Exercise 8 - Run tool Checkov in PR and integrate in Code Scanning](#exercise-8---run-tool-checkov-in-pr-and-integrate-in-code-scanning-15-mins)
- [ ] [Extra - CodeQL workflow optimization](#extra---codeql-workflow-optimization-10-mins)
- [ ] [Exercise 9 -  Integrate Secret Scanning alerts in PR](#exercise-9---integrate-secret-scanning-alerts-in-pr-10-mins)
- [ ] [Exercise 10 - Generate CodeQL debug and identify a problem](#exercise-10---generate-codeql-debug-and-identify-a-problem-15-mins)

## How does it work? (Contd.)

### Exercise 7 - Customizing CodeQL Configuration (15 mins)

By default, CodeQL uses a selection of queries that provide high quality security results.
However, you might want to change this behavior to:

- Include code-quality queries.
- Include queries with a lower signal to noise ratio to detect more potential issues.
- To exclude queries in the default pack because they generate *false positives* for your architecture.
- Include custom queries written for your project.

1.  Create the file `.github/codeql/codeql-config.yml` and enable the `security-and-quality` suite.

    **Hints**

    1. A configuration file contains a key `queries` where you can specify additional queries as follows

        ```yaml
        name: "My CodeQL config"

        queries:
            - uses: <insert your query suite>
        ```
2. Enable your custom configuration in the code scanning workflow file `.github/codeql/codeql-config.yml`

    **Hints**

    1. The `init` action supports a `config-file` parameter to specify a configuration file.

3. After the code scanning action has completed, are there new code scanning results?

### Adding your own code scanning suite to exclude rules

The queries that are executed is determined by the code scanning suite for a target language.
You can create your own code scanning suite to change the set of included queries.

By creating our own [code scanning suite](https://codeql.github.com/docs/codeql-cli/creating-codeql-query-suites/), we can exclude the rule that caused the false positive in our Java project.

1. Create the file `custom-queries/code-scanning.qls` with the contents

    ```yaml
    # Reusing existing QL Pack
    - import: codeql-suites/javascript-code-scanning.qls
      from: codeql-javascript
    - import: codeql-suites/java-code-scanning.qls
      from: codeql-java
    - import: codeql-suites/python-code-scanning.qls
      from: codeql-python
    - import: codeql-suites/go-code-scanning.qls
      from: codeql-go
    - exclude:
        id:
        - <insert rule id of false positive>
    ```

2. Configure the file `.github/codeql/codeql-config.yml` to use our suite.

    **Hint**: We are now running both the default code scanning suite and our own custom suite.
    To prevent CodeQL from resolving queries twice, disable the default queries with the option `disable-default-queries: true`

<details>
<summary>Solution</summary>

```yaml
name: "My CodeQL config"

disable-default-queries: true

queries:
    - uses: ./custom-queries/code-scanning.qls
```
</details>

3. After the code scanning action has completed, is the false positive still there?

4. Try running additional queries with `security-extended` or `security-and-quality`. What kind of results do you see?

**Note**: If you want to use these additional query suites and the custom query suite you've made, make sure to import the proper query packs to continue to exclude certain queries.

<details>
<summary>Solution</summary>

```yaml
# Reusing existing QL Pack
- import: codeql-suites/javascript-security-and-quality.qls
  from: codeql-javascript
- import: codeql-suites/java-security-and-quality.qls
  from: codeql-java
- import: codeql-suites/python-security-and-quality.qls
  from: codeql-python
- import: codeql-suites/go-security-and-quality.qls
  from: codeql-go
- exclude:
  id:
    - java/spring-disabled-csrf-protection
```
</details>

5. Try specifying directories to scan or not to scan. Note that this is only supported for interpreted languages, such as javascript/typescript, python, ruby, etc. Why would you include this in the configuration?

<details>
<summary>Solution</summary>

```yaml
name: "My CodeQL config"

disable-default-queries: true

queries:
    - uses: ./custom-queries/code-scanning.qls

paths-ignore:
 - '**/test/**'
```
</details>

### Understanding how to add a custom query

One of the strong suites of CodeQL is its high-level language QL that can be used to write your own queries.
_If you have experience with CodeQL and have come up with your own query so far, take this time to commit those changes and see if any alerts were produced._
Regardless of experience, the next steps show you how to add one.

1. Make sure to create a QL pack file. For example, `custom-queries/go/qlpack.yml` with the contents

    ```yaml
    name: my-go-queries
    version: 0.0.0
    libraryPathDependencies:
        - codeql-go
    ```

    This file creates a [QL query pack](https://help.semmle.com/codeql/codeql-cli/reference/qlpack-overview.html) used to organize query files and their dependencies.

2. Then, create the actual query file. For example, `custom-queries/go/jwt.ql` with the contents

    ```ql
    /**
    * @name Missing token verification
    * @description Missing token verification
    * @id go/jwt-sign-check
    * @kind problem
    * @problem.severity warning
    * @precision high
    * @tags security
    */
    import go
    /*
    * Identify processors that are missing the token verification:
    *
    * func(token *jwt.Token) (interface{}, error) {
    *    // Don't forget to validate the alg is what you expect:
    *    //if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
    *    //        return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
    *    //}
    *    ...
    * }
    */
    from FuncLit f
    where
        // Identify the function via the argument part of the its signature
        //     func(token *jwt.Token) (interface{}, error) { ... }
        f.getParameter(0).getType() instanceof PointerType and
        f.getParameter(0).getType().(PointerType).getBaseType().getName() = "Token" and
        f.getParameter(0).getType().(PointerType).getBaseType().getPackage().getName() = "jwt" and
        // and check whether it uses jwt.SigningMethodHMAC in any way
        not exists(TypeExpr t |
            f.getBody().getAChild*() = t and
            t.getType().getName() = "SigningMethodHMAC" and
            t.getType().getPackage().getName() = "jwt"
        )
    select f, "This function should be using jwt.SigningMethodHMAC"
    ```
3. Then, add the query to the CodeQL configuration file `.github/codeql/codeql-config.yml`

**Hint** The `uses` key accepts repository relative paths.

<details>
<summary>Solution</summary>

```yaml
name: "My CodeQL config"

disable-default-queries: true

queries:
    - uses: security-and-quality
    - uses: ./custom-queries/code-scanning.qls
    - uses: ./custom-queries/go/jwt.ql

```
</details>

### _Stretch Exercise: Adding a custom query from an external repository_

How would you incorporate that query/queries from other repositories?

<details>
<summary>Solution</summary>

```yaml
name: "CodeQL Config"

disable-default-queries: false

queries:
  - name: go-custom-queries
    uses: {owner}/{repository}/<path-to-query>@<some-branch>
  - uses: security-and-quality
```
</details>

### Extra - CodeQL workflow optimization (10 mins)

In this exercise, we will optimize the CodeQL workflow we created to run only when code changes happen and shorten the runtime on. 

To do this you will need to:

- Add a `paths` and `paths-ignore` filter to the workflow to only run when code changes happen
- Add a `cache` step to the workflow to cache the dependencies between runs.

Using the information provided on [customizing advanced setup](https://docs.github.com/en/enterprise-cloud@latest/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning) change the triggers for your Codeql workflow to not run when `.md` and `.txt` files are changed.
We also want to only trigger the workflow when code changes in `authn-service`, `frontend`, `gallery-service` and `storage-service` happen. To do this, we will use the `paths` and `paths-ignore` filters.

<details>
<summary>Solution</summary>

```yaml
name: "CodeQL"

on:
  push:
    branches: [ main ]
    paths:
      - 'authn-service/**'
      - 'frontend/**'
      - 'gallery-service/**'
      - 'storage-service/**'
    paths-ignore:
      - '**.md'
      - '**.txt'
  pull_request:
    paths:
      - 'authn-service/**'
      - 'frontend/**'
      - 'gallery-service/**'
      - 'storage-service/**'
    paths-ignore:
      - '**.md'
      - '**.txt'
```
</details>

Next, we will add a `cache` steps to the workflow to cache the dependencies between runs for the Java and Go components. This will speed up the workflow execution time. To do this, you can use the `actions/setup-java` and `actions/setup-go` actions.

As you have learned before, you can only condition these setups to the appropriate jobs. To do this, you can use the `if` conditional on the job level.

<details>
<summary>Solution</summary>

```yaml
- if: matrix.language == 'java-kotlin'
  name: Setup Java
  uses: actions/setup-java@v3
  with:
    distribution: 'temurin'
    java-version: '17'
    cache: 'maven'

- if: matrix.language == 'go'
  name: Setup Go
  uses: actions/setup-go@v3
  with:
    go-version: '1.17'
    cache: true
    cache-dependency-path: gallery-service/go.sum
```

</details>

## Integrations - GHAS API and Webhooks

### Exercise 8 - Run tool Checkov in PR and integrate in Code Scanning (15 mins)

In this exercise, we will integrate another security testing tool's results into Code Scanning. The choice of a tool is Checkov. We will create a new GitHub Action workflow where we will run Checkov scan against our codebase and report back the results to GitHub by calling the Code Scanning API.

### Create a new workflow

You need to create a new workflow file that will do the analysis. To do this, you can navigate to the `Actions` tab of your repository and select `New workflow`. You will be presented with a list of templates.
We will not be using already existing workflow on the marketplace in this exercise, so you will need to select `Skip this and set up a workflow yourself`.

Navigate to [bridgecrewio/checkov-action](https://github.com/bridgecrewio/checkov-action) and copy the example config. Rename the workflow file to `checkov.yml` and paste the content into the editor and commit the file to your repository.

<details>
<summary>Solution</summary>

```yaml
name: checkov

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the "main" branch
  push:
    branches: [ "main", "master" ]
  pull_request:
    branches: [ "main", "master" ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "scan"
  scan:
    permissions:
      contents: read # for actions/checkout to fetch code
      security-events: write # for github/codeql-action/upload-sarif to upload SARIF results
      actions: read # only required for a private repository by github/codeql-action/upload-sarif to get the Action run status
      
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so follow-up steps can access it
      - uses: actions/checkout@v4

      - name: Checkov GitHub Action
        uses: bridgecrewio/checkov-action@v12
        with:
          # This will add both a CLI output to the console and create a results.sarif file
          output_format: cli,sarif
          output_file_path: console,results.sarif
        
      - name: Upload SARIF file
        uses: github/codeql-action/upload-sarif@v2
        
        # Results are generated only on a success or failure
        # this is required since GitHub by default won't run the next step
        # when the previous one has failed. Security checks that do not pass will 'fail'.
        # An alternative is to add `continue-on-error: true` to the previous step
        # Or 'soft_fail: true' to checkov.
        if: success() || failure()
        with:
          sarif_file: results.sarif
```
</details>

After committing the file you should see a new workflow running in the `Actions` tab. 

### Trigger a new checkov run

To trigger a new checkov run you can create a new branch and edit the `terraform/aws/lambda.tf` file and add the following snippet:

```terraform
resource "aws_iam_role" "iam_for_lambda" {
  name = "${local.resource_prefix.value}-analysis-lambda"


  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
  tags = {
    git_commit           = "e6d83b21346fe85d4fe28b16c0b2f1e0662eb1d7"
    git_file             = "terraform/aws/lambda.tf"
    git_last_modified_at = "2023-04-27 12:47:51"
    git_last_modified_by = "nadler@paloaltonetworks.com"
    git_modifiers        = "nadler/nimrodkor"
    git_org              = "bridgecrewio"
    git_repo             = "terragoat"
    yor_trace            = "93cfa6f9-a257-40c3-b7dc-3c3686929734"
  }
}

resource "aws_lambda_function" "analysis_lambda" {
  # lambda have plain text secrets in environment variables
  filename      = "resources/lambda_function_payload.zip"
  function_name = "${local.resource_prefix.value}-analysis"
  role          = "${aws_iam_role.iam_for_lambda.arn}"
  handler       = "exports.test"

  source_code_hash = "${filebase64sha256("resources/lambda_function_payload.zip")}"

  runtime = "nodejs13.x"

  environment {
    variables = {
      access_key = "AKIAIOSFODNN7EXAMPLE"
      secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
    }
  }
  tags = {
    git_commit           = "5c6b5d60a8aa63a5d37e60f15185d13a967f0542"
    git_file             = "terraform/aws/lambda.tf"
    git_last_modified_at = "2021-05-02 10:06:10"
    git_last_modified_by = "nimrodkor@users.noreply.github.com"
    git_modifiers        = "nimrodkor"
    git_org              = "bridgecrewio"
    git_repo             = "terragoat"
    yor_trace            = "f7d8bc47-e5d9-4b09-9d8f-e7b9724d826e"
  }
}
```

Next, go ahead and raise a PR against the default branch and check if that will trigger an analysis.

---

### Exercise 9 -  Integrate Secret Scanning alerts in PR (10 mins)

Similar to the previous Exercise we will now integrate the [advanced-security/secret-scanning-action](https://github.com/advanced-security/secret-scanning-review-action). This time we will not be doing any local static checks against a codebase in the workflow but rather fetch information from the available GHAS APIs and display the results in the PR.

### Create a new workflow

Following the same steps, create a new workflow file that will run the [advanced-security/secret-scanning-review-action](https://github.com/advanced-security/secret-scanning-review-action) action.

As also pointed out in the `README` of the `advanced-security/secret-scanning-review-action`, you will have to create a new GitHub token with the `repo` scope and add it as a secret to your repository with a name `SECRET_SCAN_REVIEW_GITHUB_TOKEN`.

<details>
<summary>Solution</summary>

```yaml
name: 'Secret Scanning Review'
on: [pull_request]

jobs:
  secret-scanning-review:
    runs-on: ubuntu-latest
    steps:
      - name: 'Secret Scanning Review Action'
        uses: advanced-security/secret-scanning-review-action@v0
        with:
          token: ${{ secrets.SECRET_SCAN_REVIEW_GITHUB_TOKEN }}
          fail-on-alert: true
          fail-on-alert-exclude-closed: true
```
</details>

#### Trigger an action run in PR

To trigger a new checkov run you can create a new branch and edit the `authn-service/.env` file and add the following secret `STRIPE_DEV="sk_live_devboxbcct1DfwS2ClCIKlbT"`
Since you have Push Protection enabled go ahead and bypass it with `I'll fix it later` reason.

Raise a PR and check if that will trigger an analysis and observe the results.

---

### Exercise 10 - Generate CodeQL debug and identify a problem (15 mins)

In this exercise, we will generate and look at options for you to debug a Code Scanning - CodeQL debug analysis run. We will generate a CodeQL Code Scanning debug run file and understand the structure of a debug artifact. Then, we will also look at a few debug artifacts to identify the problems and, lastly, how we can use the `Tool Status` to understand the status of the CodeQL analysis.

### Enabling CodeQL Debug mode

First, navigate to the Actions tab on your repository and select the `CodeQL` workflow.

You can re-run a CodeQL analysis in debug mode by selecting a CodeQL analysis workflow run, then clicking `Re-run all jobs` in the top right of the page and selecting `Enable debug logging` before hitting the `Re-run jobs` button. This will trigger a new CodeQL analysis run after which the debug data will be uploaded as Actions artifact. After the run has completed, you should see a list of artifacts `debug-artifacts-*`. Go ahead and download the `debug-artifacts-java`.

The other option to enable CodeQL debug mode in the Action is by providing `debug: true` configuration option. This is useful when you want to enable debug mode for a workflow where you expect it to fail, and you don't want to wait for it to finish before re-running it in debug mode. The end result is the same.

### Understanding the debug artifact

The debug artifact is a zip file that contains a number of files and directories. The most important ones are:

- `java/log/`
  - `ext/*` - a directory containing the CodeQL extractor configuration, environment variables, etc.
  - `log/database-*` - files containing logs from the CodeQL database-related operations - creation, initialization, etc.
  - `log/javac-*` - files containing logs from the compilation and extractor process.
- `log` - logs from the build tracker and database initialization.
- `java.sarif` - the results of the analysis in SARIF format.
- `db-java.zip` - the CodeQL database that was created and used for the analysis. This is the most important file as it contains all the information about the analysis. Be aware when sharing this archive as it also contains the source code of your repository.

### Investigating the CodeQL debug artifact

In this exercise, we will look at a few debug artifacts and try to identify the problem.

Go ahead and download the debug artifacts from the following link: [code-debug-artifacts-java](https://gh.io/ghas-training-debug-artifacts)

Using the artifact provided, can you answer the following questions:

- Which CodeQL CLI version was used for the analysis?
- What type of codebase was the analysis run on?
- Was the analysis completed successfully? If not, what was the reason?
- Can you identify which CodeQL queries were configured for this analysis?

Hint: Look into the `db-cpp-partial.zip` file

<details>
<summary>Solution</summary>
- Which CodeQL CLI version was used for the analysis?
    - `CodeQL CLI version: 2.15.1`
- What type of codebase was the analysis run on?
    - `C or Cpp`, as can be guessed from the filename, but also the extractor used, or the `codeql-database.yml` file.
- Was the analysis completed successfully? If not, what was the reason?
    - No, the analysis failed because `No supported build system detected`, as it can be seen in the extractor diagnostic message.
    - Digging further in, we can also see that there were indeed empty source directories without code or build systems.
- The files `config-queries.qls` give us the list of queries that were configured for this analysis.
</details>

### Using Tool Status

In the `Security` tab of your repository, under Code Scanning, you have a section called `Tool status`. This is a good place to check if there are any issues with the CodeQL analysis as well as find information about the CodeQL version used for the analysis, CodeQL queries used, etc.

Navigate to the `Tool status` section and answer the following questions:

- Which CodeQL CLI version was used for the analysis?
- Which Query Suite was used for the analysis for the last analysis of the Java codebase?
- How many rules were ran in the last analysis of the Python codebase?
- Were all the files from the Go component `gallery-service` scanned?