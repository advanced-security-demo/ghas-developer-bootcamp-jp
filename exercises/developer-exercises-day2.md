# GHAS for platform engineerrs

## Overview

- [ ] Exercise 7 - CodeQL構成のカスタマイズ
- [ ] 番外編 - CodeQLワークフローの最適化
- [ ] Exercise 8 - PRでツールCheckovを実行し、Code Scanningへ統合
- [ ] Exercise 9 -  PRへSecret Scanning alertsを統合
- [ ] Exercise 10 - CodeQLデバッグの生成と問題の特定

## How does it work? (続き)

### Exercise 7 - CodeQL構成のカスタマイズ (15分)

デフォルトでは、CodeQLは高品質のセキュリティ結果を提供するクエリを選択して使用します。
ただし、この動作を次のように変更することもできます：

- コード品質のクエリーを含める。
- より多くの潜在的な問題を検出するために、S/N比の低い（lower signal to noise ratio）クエリを含める。
- デフォルトのパックに含まれるクエリは、アーキテクチャに対して偽陽性（false positive）を生成するため、除外する。
- プロジェクト用に作成されたカスタムクエリを含める。

1.  `.github/codeql/codeql-config.yml` ファイルを作成し、`security-and-quality` スイートを有効にします.

    **Hints**

    1. 設定ファイルには `queries` というキーがあり、以下のように追加のクエリを指定することができます。

        ```yaml
        name: "My CodeQL config"

        queries:
            - uses: <insert your query suite>
        ```
2. Code scanning ワークフローファイル `.github/codeql/codeql-config.yml` でカスタム設定を有効にします。

    **Hints**

    1. `init` アクションは設定ファイルを指定するための `config-file` パラメータをサポートしています。

3. コードスキャンが完了した後、新しいコードスキャン結果は存在するでしょうか。

### 除外ルールに独自のコードスキャンスイートを追加

実行されるクエリーは、ターゲット言語のコードスキャンスイートによって決定されます。
独自のコードスキャンスイートを作成して、含まれるクエリーのセットを変更することができます。

独自の[コードスキャンスイート](https://codeql.github.com/docs/codeql-cli/creating-codeql-query-suites/)を作成することで、Javaプロジェクトで誤検出を引き起こしたルールを除外することができます。

1. `custom-queries/code-scanning.qls` ファイルを下記のコンテンツを含んで作成します。

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

2. `.github/codeql/codeql-config.yml` ファイルをスイートに構成します。

    **Hint**: 現在、デフォルトのコードスキャンスイートと独自のカスタムスイートの両方を実行しています。
    CodeQLがクエリーを2回解決しないようにするには、`disable-default-queries: true`オプションでデフォルトのクエリーを無効にします。

<details>
<summary>Solution</summary>

```yaml
name: "My CodeQL config"

disable-default-queries: true

queries:
    - uses: ./custom-queries/code-scanning.qls
```
</details>

3. コードスキャンが完了した後、誤検出はまだありますか。

4. `security-extended`または`security-and-quality`で追加のクエリを実行してみてください。どのような結果が出ますか？

**注意***： これらの追加クエリスイートや作成したカスタムクエリスイートを使用したい場合は、特定のクエリを除外し続けるために適切なクエリパックをインポートしてください。

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

5. スキャンするディレクトリとしないディレクトリを指定してみてください。これは、javascript/typescript、python、rubyなどのインタプリタ型言語でのみサポートされていることに注意してください。なぜこれを設定に含めるのでしょうか。

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

### カスタムクエリの追加方法を理解する

CodeQLの強力なスイートの1つは、独自のクエリを書くために使用できる高レベル言語QLです。
_CodeQLを使用した経験があり、これまでに独自のクエリを作成したことがある場合は、この機会にそれらの変更をコミットし、アラートが生成されたかどうかを確認してください。_
経験の有無にかかわらず、次のステップではアラートを追加する方法を説明します。

1. QLパックファイルを必ず作成してください。例えば、`custom-queries/go/qlpack.yml`とします。

    ```yaml
    name: my-go-queries
    version: 0.0.0
    libraryPathDependencies:
        - codeql-go
    ```

    このファイルは、クエリーファイルとその依存関係を整理するために使用される[QLクエリーパック](https://help.semmle.com/codeql/codeql-cli/reference/qlpack-overview.html)を作成します。

2. クエリーファイルを作成します。例えば下記のコンテンツを含む `custom-queries/go/jwt.ql` ファイルを作成します。

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
3. 次に、CodeQL設定ファイル`.github/codeql/codeql-config.yml`にクエリを追加します。

**Hint**  `uses` キーはリポジトリの相対パスを受け付けます。

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

他のリポジトリからのクエリをどのように組み込むのでしょうか。

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

### 番外編 - CodeQLワークフローの最適化 (10分)

このエクササイズでは、作成したCodeQLワークフローをコード変更時にのみ実行するように最適化し、実行時間を短縮します。

そのためには以下のことが必要です：

- ワークフローに `paths` と `paths-ignore` フィルタを追加し、コード変更時にのみ実行するようにする。
- ワークフローに `cache` ステップを追加し、実行時に依存関係をキャッシュする。

[高度なセットアップのカスタマイズ](https://docs.github.com/en/enterprise-cloud@latest/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning)で提供されている情報を使用して、`.md`ファイルと`.txt`ファイルが変更されたときに実行されないように、Codeqlワークフローのトリガーを変更します。
また、`authn-service`、`frontend`、`gallery-service`、`storage-service`のコードが変更されたときのみワークフローを起動するようにするためには、 `paths` フィルタと `paths-ignore` フィルタを使用します。

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

次に、ワークフローに `cache` ステップを追加し、Java と Go コンポーネントの実行間の依存関係をキャッシュします。これにより、ワークフローの実行時間が短縮されます。これを行うには、`actions/setup-java` アクションと `actions/setup-go` アクションを使用します。

前に学んだように、これらのセットアップを適切なジョブにのみ適用することができます。これを行うには、ジョブレベルで `if` 条件を使用します。

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

### Exercise 8 - PRでツールCheckovを実行し、Code Scanningへ統合  (15分)

この演習では、別のセキュリティテストツールの結果をCode Scanningに統合します。選んだツールは Checkov です。新しい GitHub Action ワークフローを作成し、そこでコードベースに対して Checkov スキャンを実行し、Code Scanning API を呼び出して結果を GitHub に報告します。

### ワークフローの新規作成

分析を行う新しいワークフローファイルを作成する必要があります。これを行うには、リポジトリの `Actions` タブに移動し、`New workflow` を選択します。テンプレートのリストが表示されます。
この演習ではマーケットプレイスにある既存のワークフローは使用しませんので、`Skip this and set up a workflow yourself`を選択する必要があります。

[bridgecrewio/checkov-action](https://github.com/bridgecrewio/checkov-action)に移動し、サンプルのコンフィグをコピーします。ワークフローファイルの名前を `checkov.yml` に変更し、内容をエディタに貼り付けて、ファイルをリポジトリにコミットします。

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

ファイルをコミットすると、`Actions`タブに新しいワークフローが表示されます。

### checkov runをトリガー

新しいcheckovの実行をトリガーするには、新しいブランチを作成して`terraform/aws/lambda.tf`ファイルを編集し、以下のスニペットを追加します：

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

次に、デフォルトのブランチに対してPRを作成し、それが分析をトリガーするかどうかを確認します。

---

### Exercise 9 -  PRへSecret Scanning alertsを統合 (10分)

前回のExerciseと同様に、今回は[advanced-security/secret-scanning-action](https://github.com/advanced-security/secret-scanning-review-action)を統合します。今回は、ワークフローでコードベースに対してローカルの静的チェックを行うのではなく、利用可能なGHAS APIから情報を取得し、その結果をPRに表示します。

### ワークフローの新規作成

同じ手順で、[advanced-security/secret-scanning-review-action](https://github.com/advanced-security/secret-scanning-review-action) アクションを実行する新しいワークフローファイルを作成します。

`advanced-security/secret-scanning-review-action` の `README` にもあるように、`repo` スコープで新しい GitHub トークンを作成し、`SECRET_SCAN_REVIEW_GITHUB_TOKEN` という名前でリポジトリにシークレットとして追加します。

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

#### PRでのaction run のトリガー

新しいcheckovの実行をトリガーするには、新しいブランチを作成し、`authn-service/.env`ファイルを編集して、次のシークレット`STRIPE_DEV="sk_live_devboxbcct1DfwS2ClCIKlbT"`
を追加します。
プッシュプロテクションが有効になっているので、`I'll fix it later`を理由にプッシュプロテクションをバイパスします。

PRを発行し、分析がトリガーされるかどうかを確認し、結果を観察してください。

---

### Exercise 10 - CodeQLデバッグの生成と問題の特定 (15分)

この実習では、Code Scanning - CodeQL debug analysis runをデバッグするためのオプションを生成し、見ていきます。CodeQL Code Scanningデバッグ実行ファイルを生成し、デバッグ成果物の構造を理解します。また、いくつかのデバッグ成果物を見て問題を特定し、最後に`Tool Status`を使用してCodeQL解析のステータスを理解します。

### CodeQLデバッグモードの有効化

まず、リポジトリのActionsタブに移動し、`CodeQL`ワークフローを選択します。

デバッグモードでCodeQL分析を再実行するには、CodeQL分析ワークフローの実行を選択し、ページ右上の`Re-run all jobs`をクリックし、`Re-run jobs`ボタンを押す前に`Enable debug logging`を選択します。これにより、新しいCodeQL分析が実行され、デバッグデータがアクションアーティファクトとしてアップロードされます。実行が完了すると、アーティファクトのリスト `debug-artifacts-*` が表示されます。先に進み、`debug-artifacts-java`をダウンロードします。

ActionでCodeQLのデバッグモードを有効にするもう1つの方法は、`debug: true`設定オプションを指定することです。これは、失敗することが予想されるワークフローでデバッグモードを有効にし、デバッグモードで再実行する前に終了するのを待ちたくない場合に便利です。最終的な結果は同じです。

### デバッグアーティファクトについて

デバッグアーティファクトはZIPファイルであり、いくつかのファイルとディレクトリが含まれています。最も重要なものは下記のとおりです。

- `java/log/`
  - `ext/*` - CodeQLエクストラクタの設定、環境変数などを含むディレクトリ
  - `log/database-*` - CodeQLデータベース関連の操作（作成、初期化など）のログを含むファイル
  - `log/javac-*` - コンパイルおよび抽出処理からのログを含むファイル
- `log` - ビルドトラッカーとデータベースの初期化のログを含むファイル
- `java.sarif` - SARIF 形式の解析結果
- `db-java.zip` - 分析に作成＆使用されたCodeQLデータベース。分析に関するすべての情報が含まれているため、最も重要なファイルです。リポジトリのソースコードも含まれているため、このアーカイブを共有する際には注意してください。

### CodeQLデバッグアーティファクトの調査

この実習では、いくつかのデバッグアーティファクトを見て、問題の特定を試みます。

以下のリンクからデバッグアーティファクトをダウンロードしてください： [code-debug-artifacts-java](https://gh.io/ghas-training-debug-artifacts)

提供されたアーティファクトを使用して、以下の質問に答えることができますか：

- 解析に使用したCodeQL CLIのバージョンは。
- 解析はどのようなコードベースで実行されましたか。
- 解析は正常に完了しましたか？完了しなかった場合、その理由は何ですか。
- この解析のためにどの CodeQL クエリーが構成されたかを特定できますか。

ヒント：`db-cpp-partial.zip`ファイルを見てください。

<details>
<summary>Solution</summary>
- 解析に使用したCodeQL CLIのバージョンは。
    - `CodeQL CLI version: 2.15.1`
- 解析はどのようなコードベースで実行されましたか。
    - `C or Cpp`, as can be guessed from the filename, but also the extractor used, or the `codeql-database.yml` file.
- 解析は正常に完了しましたか？完了しなかった場合、その理由は何ですか。
    - No, the analysis failed because `No supported build system detected`, as it can be seen in the extractor diagnostic message.
    - Digging further in, we can also see that there were indeed empty source directories without code or build systems.
- この解析のためにどの CodeQL クエリーが構成されたかを特定できますか。
    - The files `config-queries.qls` give us the list of queries that were configured for this analysis.
</details>

### Tool Statusの利用

リポジトリの `Security` タブの Code Scanning の下に、`Tool status` というセクションがあります。これは、CodeQL分析に問題があるかどうかを確認したり、分析に使用されたCodeQLのバージョンや使用されたCodeQLクエリなどの情報を見つけるのに適した場所です。

`Tool status`セクションに移動し、以下の質問に答えてみてください：

- 解析に使用したCodeQL CLIのバージョンは。
- Javaコードベースの最後の解析では、どのQuery Suiteが解析に使用されましたか。
- Pythonコードベースの最後の解析ではいくつのルールが実行されましたか。
- Goコンポーネント`gallery-service`のファイルはすべてスキャンされましたか。


