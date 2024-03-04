# GHAS for developers

## Overview

- [ ] Exercise 1 - リポジトリのGHAS有効化
- [ ] Exercise 2 - Dependabot
- [ ] Exercise 3 - Dependancy Review
- [ ] Exercise 4 - Secret Scanning と Push Protection
- [ ] Exercise 5 - Secret Scanning - Custom Secret Scanning Pattern + Push protection
- [ ] Exercise 6 - Code Scanning と CodeQL

## GHAS Enablement

### Exercise 1 - リポジトリのGHAS有効化 (10分)

### Dependabot alertsの有効化

Dependabot は、Organizationまたはリポジトリの設定で有効にすることができます。

- リポジトリの設定に行き、`Code security and analysis` セクションで Dependabot alerts を有効にしてください。dependency graphがまだ有効になっていない場合は、有効にするようプロンプトが表示されます。

### Dependency graph（依存関係グラフ）のレビュー

- 依存関係グラフで、以下の依存関係が見つかったことを確認しましょう：
    - frontend service
    - authentication service
    - gallery service
    - storage service

依存関係グラフは、リポジトリの `Insights` タブからアクセスできます。

### 結果の閲覧と管理

数分後、リポジトリの `Security` タブに、新しいセキュリティ警告があることが表示されます。このボタンをクリックすると、脆弱な依存関係を更新するプルリクエスト (PR) が作成されます。このボタンをクリックすると、脆弱性のある依存関係を更新するプルリクエスト (PR) が作成されます。次のセクションでは、該当するすべての Dependabot アラートのセキュリティ更新を有効にする方法を説明します。

**注意 次のセクションでは、該当する Dependabot アラートのセキュリティアップデートを有効にする方法を説明します。

1. Dependabot アラート セクションに移動して、検出された依存関係の問題を表示します。

各依存性アラートに対して、セキュリティ更新プログラムを作成するか、理由とともにアラートを解除するオプションがあります。

2. アラートの 1 つについて、依存関係のセキュリティ更新を作成します。Dependabot が依存関係を自動的に更新できる場合、PR を作成します。

3. アラートの 1 つについて、アラートを却下します。

### Dependabot security updatesの有効化

Dependabot は、脆弱な依存関係を脆弱でないバージョンにアップグレードするための PR を自動的に作成することができます。Dependabot のアラートの中には、パッチがないものがあるかもしれないことに注意してください。そのような場合、セキュリティアップデートは利用できません。

- リポジトリ設定に移動し、*Code security & analysis*セクションでDependabot security updatesを有効にしてください。

数分後、脆弱な依存関係をアップグレードする複数の PR が作成されます。

### GitHub Adanced securityの有効化

#### リポジトリ　レベル

これまで GHAS ライセンスは消費されていませんでした。それでは、GitHub Advanced Security を有効にしましょう。リポジトリの設定の`Code security and analysis`セクションに戻ります。

すると、Dependabot オプションのすぐ下に`GitHub Advanced Security`セクションがあることに気づくでしょう。`Enable`ボタンをクリックします。この操作を確認するモーダルが表示されます。`Enable GitHub Advanced Security` ボタンをクリックして確定します。モーダルでは、消費されるライセンス数 (GitHub Advanced Security のシート数) についての情報も表示されます。消費されるシート数は、リポジトリのアクティブなコミッター数と同じです。

GitHub Advanced Security を有効にすると、`Code scanning alerts` セクションのオプションが増えたことに気づくでしょう。その前に、GitHub Advanced Security を有効にする他の方法を探ってみましょう。

#### Organization レベル

リポジトリレベルと同様に、`Code security and analysis` に移動してください。リポジトリレベルとほとんど同じように表示されます。大きな違いはありません：

- `Enable All` / `Disable All` ボタンが存在します
- チェックボックスオプション `Automatically enable for new private and internal repositories`. このオプションは、organization内で新しく作成されたPrivateリポジトリやInternalリポジトリに対して自動的に GitHub Advanced Security を有効にします。この機能を有効にすると便利ですが、GHAS ライセンスを考慮する必要があります。

注意: organizationの管理者である場合は、`Enable All` ボタンをクリックしないでください。このワークショップでは、リポジトリレベルの有効化について説明します。

## How does it work?

### Exercise 2 - Dependabot 構成 (10分)

#### Dependabot version updatesの構成

リポジトリの `.github` ディレクトリに dependabot.yml をチェックインすることで、Dependabot [*version updates*](https://docs.github.com/ja/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/enabling-and-disabling-version-updates) を有効にできます。Dependabot セキュリティアップデートは、この設定も使用します。セキュリティアップデートを SDLC にうまく統合するために、以下のようなさまざまな点を設定することができます：

- いつバージョン更新が作成されるか。
- フィルタリングオプションを有効にするために、どのようなラベルを割り当てるか。
- 誰が PR を担当し、誰がレビューすべきか。
- どの依存関係をどのように更新するか。

`.github/dependabot.yml`ファイルをリポジトリに作成し、`pip`依存性マネージャーを以下のように設定します：
  1. `authn-service` ディレクトリで依存情報を探します。

  2. 毎日のバージョン更新をスケジュールします。

  3. コミットメッセージの先頭に `pip` パッケージマネージャーを付けます。

  4. 自分自身とワークショップチームの一人をレビュアーに任命する。ymlでGitHubハンドルを指定するときは、`@`記号を使わないでください。例として以下のソリューションをご覧ください。

  5. PR のフィルタリングを有効にするために、カスタムラベル `triage-required` を追加します。(カスタムラベルは先にPR画面で作成してください。)

  6. [vulnerable dependency](https://github.com/advisories?query=severity%3Ahigh+ecosystem%3Apip)を `auth-service/requirements.txt` に追加して、変更を検証してください。例えば

    ```requirements.txt
    ...
    django==2.1.0
    ```

コンフィギュレーションが満足できないかどうかは、どのように判断するのでしょうか。

1. コンフィギュレーションに存在しないラベルを追加します。

2. プロジェクトの1つに脆弱な依存関係を追加することで、新しい dependabot セキュリティ更新をトリガーします。
   例えば、依存関係 `django-two-factor-auth==1.11` を `auth-service/requirements.txt` に追加します。

3. 作成された PR を見て、設定が満たされているかどうかを判断します。

4. Dependabot チャットコマンドを使って [Dependabot から PR を管理する](https://docs.github.com/ja/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/managing-pull-requests-for-dependency-updates).

<details>
<summary>Solution</summary>

```yaml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/authn-service"
    schedule:
      interval: "daily"
    labels:
      - "triage-required"
    assignees:
      - "dungeonstechlead"
    reviewers:
      - "dragonsengineer"
    commit-message:
      prefix: "pip"
```
</details>

### _Stretch_ Grouping Dependabot PRs

2023年6月に[group version updates](https://github.blog/changelog/2023-06-30-grouped-version-updates-for-dependabot-public-beta/) をPRにまとめる方法を開始しました。

これは今までと同じ `dependabot.yml` ファイルを使って設定することができます。以下に例を示します：

```yaml
version: 2
  updates:
  - package-ecosystem: "bundler"
    directory: "/"
    schedule:
      interval: weekly
    # New!
    groups:
      # This is the name of your group, it will be used in PR titles and branch names
      dev-dependencies:
        # A pattern can be...
        patterns:
          - "rubocop" # a single dependency name
          - "aws*"  # or a wildcard string that matches multiple dependencies
          # If you'd simply like to group as many dependencies together as possible, 
          # you can use the wildcard * - but keep in mind this may open a very large PR!
        # Additionally, you can specify any dependencies to be excluded from the group
        exclude-patterns:
          - "aws-sdk"
```

設定オプション `groups` の詳細については [オンラインドキュメント](https://docs.github.com/en/enterprise-cloud@latest/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file) を参照してください。

dependabot.ymlの設定を更新して、今後すべての `npm` 開発用依存関係を `dev-dependencies` というPRにグループ化するにはどうすればいいかも考えて見ましょう。

### Exercise 3 - Dependancy Review (10分)

### ワークフローの作成

1. https://github.com/actions/dependency-review-action を参照し、[docs/examples.md](https://github.com/actions/dependency-review-action/blob/main/docs/examples.md#basic-usage)からBasic Usageワークフローをコピーします。
2. `.'github/workflows` にファイルを作成します。ファイル名を `dependancy-review.yml` とします。
3. ワークフローファイルの内容を貼り付け、デフォルトのBranch `main` に保存します。

### PRでActionをトリガーする

これで Dependancy Review アクションがリポジトリに追加されました。テストするには；

1. https://github.com/advisories?query=type%3Areviewed+ecosystem%3Apip に移動します。
2. 深刻度 `high` 以上の pip の脆弱な依存関係を選択します。例：https://github.com/advisories/GHSA-rwmf-w63j-p7gv
3. `authn-service/requiremnts.txt` ファイルを編集し、新しい脆弱な依存関係を追加します。
4. **コミットする際は、新しいブランチにコミットしてください。**
5. メインブランチ `main` に対して PR をオープンし、結果を観察します。

<details>

<summary>Hints</summary>

Basic usage workflow:

```yaml
name: 'Dependency Review'
on: [pull_request]

permissions:
  contents: read

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout Repository'
        uses: actions/checkout@v4
      - name: 'Dependency Review'
        uses: actions/dependency-review-action@v3
```

Example vulnerable dependancy:


`authn-service/requirements.txt`
```text
...
cairosvg=2.6.0
```

</details>



### カスタム構成

次に、例えば下記のポリシーに合うように設定をカスタマイズしたい：

- 深刻度 `high` 以上の脆弱性に対しては失敗する。
- `BSD-2-Clause` と `LGPL-2.0` ライセンスの依存関係のみを許可する。

また、PR に直接チェックの結果を表示することで、より良いエクスペリエンスを提供します。

利用可能な設定パラメータを Dependancy Review アクションリポジトリの README で確認してください。`fail-on-severity`、`allow-license`、`comment-summary-in-pr` オプションがあります。これらを使用してワークフローファイルを調整し、結果をテストしてください。

<details>
<summary>Solution</summary>

`.github/workflows/dependancy-review.yml`

```yaml
name: 'Dependency Review'

on: 
  pull_request:
    types: [opened, reopened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout Repository'
        uses: actions/checkout@v4
      - name: 'Dependency Review'
        uses: actions/dependency-review-action@v3
        with:
          fail-on-severity: high
          comment-summary-in-pr: true
```

</details>

### Exercise 4 - Secret Scanning と Push Protection (10分)

### Secret scanningの有効化

Secret scanningは、組織またはリポジトリの設定で有効にできます。Advanced Securityがまだ有効になっていない場合は、まずそれを有効にしてください（同じ設定画面）。

1. リポジトリ設定に移動し、`Secret scanning`セクションでSecret scanningを有効にします。

注: まだ `Push protection` を有効にしないでください。

### Secret scanning 結果の表示と管理

数分後、リポジトリの `Security` タブに新しいセキュリティ警告があることが表示されます。

- 検出されたシークレットを見るには、`Secret scanning` セクションに移動してください。

それぞれのシークレットについて、それを閉じるためのオプションを見て、どれが最も適切かを判断してください。

### Test secretを追加して検出

テストケースを開発する際、公開しても悪用できない秘密が導入されていることに気づくかもしれません。シークレットスキャンは、このようなシークレットを検出し、警告を発します。

1. GitHub リポジトリのファイルエクスプローラーで、テスト用のシークレットを含むテストファイルを作成します。
    - 例えば、`storage-service/src/main/resources/application.dev.properties` というファイルにシークレットを記述します。
        ```
        STRIPE_NEW="sk_live_devboxbcct1DfwS2ClCIKlzP"
        ```
2. ファイルが保存されたときにシークレットが検出されるかどうかを判断します。
3. テストファイルの結果をどのように管理すべきでしょうか。

### Secret scanningからファイルを除外する

検出されたシークレットをテストで使用されているものとしてクローズすることもできますが、シークレットスキャンを設定してスキャン対象からファイルを除外することもできます。

1. `.github/secret_scanning.yml`ファイルが存在しない場合は作成してください。
2. 秘密スキャンから除外するパスのリストを追加する。パスを指定するには、[filter patterns](https://docs.github.com/en/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions#filter-pattern-cheat-sheet) を使用してください。

    ```yaml
    paths-ignore:
        - '**/test/**'
        - 'exercises/**'
    ```
**注意**： 文字 `*`、`[`、`!` パターンを `*`, `[`, `!` で始める場合は、パターンを引用符で囲む必要があります。

パターンを使って `storage-service/src/main/resources/application.dev.properties` ファイルを除外してください。

次のステップに進む前に、`.github/secret_scanning.yml` への変更をデフォルトブランチにマージしてください。

    <details>
    <summary>Solution</summary>

    ```yaml
    paths-ignore:
        - '**/test/**'
        - '**/application.dev.properties'
    ```
    </details>

3. 別のシークレットを追加するか、`storage-service/src/main/resources/application.dev.properties` ファイルに追加してパターンをテストします。

     `secretKey` のサンプル
    ```
    STRIPE_NEW="sk_live_devboxbcct1DfwS2ClCIKlbN"
    ```

### Push Protectionの有効化

`Push Protection`を有効にするには、`Code security and analysis`に戻り、`Secret Scanning`の下にある`Enable`ボタンをクリックします。

### New secretを追加して検証

先ほどと同様の手順で、Secretを含む新しいコミットをプッシュしてみましょう：

- リポジトリに新しいブランチを作成します
- UI で `authn-service/authn-service.py` を編集し、新しい secret を追加します。
        ```
        STRIPE_NEW="sk_live_devboxbcct1DfwS2ClCIKlmY"
        ```
- それでもプッシュしたい場合は、モーダルでプッシュ保護をバイパスする手順を実行する必要があります。

質問 git クライアント (あるいはお気に入りの IDE) を使ってローカルマシンから secret をプッシュしようとするとどうなるでしょうか? 

追加の練習として、次のシークレットをリポジトリにプッシュしてみてください。

```
STRIPE_NEW="sk_live_devboxbcct1DfwS2ClCIKllL"
```

---

### Exercise 5 - Secret Scanning - Custom Secret Scanning Pattern + Push protection (15分)

### Secret ScanningのためのCustom patterns

Secret Scanningは他の[シークレットパターン](https://docs.github.com/en/code-security/secret-security/defining-custom-patterns-for-secret-scanning)を見つけることをサポートします。シークレットパターンは正規表現パターンによって指定され、Hyperscanライブラリを使用します。

1. `Code security and analysis`の設定に行き、"Custom patterns"のヘッダーの下にある `New pattern`をクリックして、カスタムのシークレットパターンを追加します。
2. カスタムパターン名、シークレットフォーマット、テストケースを追加します。  例えば下記のとおりです；
```
Custom pattern name: My secret pattern
Secret format: my_custom_secret_[a-z0-9]{3}
Test string: my_custom_secret_123
```
3. パターンを保存し、Secret Scanningのアラートページでカスタムシークレットパターンが検出されたかどうかを確認します。

### Internal Tokenのためのもう一つのSecret Scanningのカスタムパターン

次のシークレットの検出に取り組んでみましょう:

```text
NBS_1: # Secret Scanning Custom Pattern  
    NBS_tkn_19vciafvzay#wa29ss15vt//tkn
    NBS_tkn_19vciuwwqaw#MM2SaXz15v//tkn
    NBS_tkn_19vrruijoaq#45qqsw115v//tkn
    NBS_tkn_19vcttijoax#4xr3zb15mx//tkn
    NBS_tkn_19vtivfzjqx#4xr3zb15mx//tkn
    NBS_tkn_19yciuijoax#4xr3zb15mx//tkn
    

Dummy value:
NBS_1: # Secret Scanning Custom Pattern  
    NBS_tkn_19vciafvzay#wa29ss15vt//tkn
    NBS_tkn_19vciuwwqaw#MM2SaXz15v//tkn
    NBS_tkn_19vrruijoaq#45qqsw115v//tkn
    NBS_tkn_19vcttijoax#4xr3zb15mx//tkn
    NBS_tkn_19vtivfzjqx#4xr3zb15mx//tkn
    NBS_tkn_19yciuijoax#4xr3zb15mx//tkn
```
<details>
<summary>Solution</summary>

```yaml
</details>
NBS_tkn_19[a-z]{9}#[a-zA-Z0-9]{10}//tkn
```

</details>

カスタムパターンを作成し、公開したら、`Push protection` チェックボックスを選択し、Push Protection スキャンにパターンを追加します。

---

### Exercise 6 - Code Scanning と CodeQL (15分)

### Code scanningの有効化

1. `Security` タブの **Vulnerability alerts** セクションで **Code scanning** をクリックし、**Configure scanning tool** ボタンをクリックします。リポジトリ設定の`Code security and analysis`セクションに移動します。このラボでは、CodeQL 関連のワークフロー ファイルを生成するために **Advanced** を選択してください。

2. 作成されたActionワークフローファイル`codeql.yml`を確認し、`Start commit`を選択してデフォルトのワークフロー案を受け入れます。

3. 作成されたワークフローを見るために、`Actions`タブに移動します。ワークフローをクリックすると、各解析ジョブの詳細とステータスが表示されます。

### 失敗した分析Jobをレビューする

CodeQL はコンパイルされた言語のビルドを必要とします。*autobuilder*が分析データベースを抽出するプログラムをビルドできない場合、分析ジョブが失敗することがあります。

1. ワークフロー内では、左側にジョブのリストが表示されます。Java ジョブをクリックしてログ出力を表示し、ビルドに失敗したかどうかを判断するためにエラーを確認します。

2. `autobuild`コンパイルの失敗は、JDKバージョンの不一致が原因のようです。私たちのプロジェクトはJDKバージョン15をターゲットにしています。GitHub がホストしているランナーが使っている Java のバージョンはどうやって確認できるでしょうか。ログの出力から何か役に立つ情報は得られますか。
詳細[runner-images](https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md)リポジトリに記載されている内容を確認するか、別の実行を実行して自分でデバッグすることができます。

    <details>
    <summary>Solution</summary>
    - GitHub はワークフローファイルをリポジトリの `.github/workflows` ディレクトリに保存します。既存の `codeql.yml` ワークフローに Java バージョンを出力するコマンドを追加することができます。 デバッグに役立つように、失敗しているステップの前のどこかに追加してください。
        
    ```yaml
    - name: Version
      run: |
        echo "java version"
        java -version
    ```
  　　　　いずれにせよ、バージョンは15以下であると結論づけられるでしょう。したがって、ビルドを成功させるためには、ランナーでJavaの正しいバージョンをセットアップする必要があります。
    </details>

3. これまでのデバッグで、ミスマッチがあると結論づけられました。 `codeql.yml`の `setup-java` アクションを使用して明示的にバージョンを指定することで、JDKのバージョンの問題を解決します。 これは `autobuild` ステップの前にワークフローに追加して、ビルド前に実行環境を適切に設定する必要があります。

    <details>
    <summary>Solution</summary>
        
    ```yaml
    - name: Java Setup
      uses: actions/setup-java@v3
      with:
        java-version: 16
        distribution: 'microsoft'
    ```
    
    </details>

### Context と Expressions を利用してビルドを修正

自動ビルドステップがコンパイル済み言語（私たちのリポジトリでは`java`）のみを対象とするように、ワークフローをどのように[修正](https://docs.github.com/en/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions)するのでしょうか。

<details>
<summary>Solution</summary>

`if` と `matrix` コンテキストを使用すると、このステップを `Java` 分析に対してのみ実行することができます。

```yaml
- if: matrix.language == 'java-kotlin'  
  uses: github/codeql-action/autobuild@v2
```
</details>

### Code scanning 結果のレビュー

1. `Security`タブで、`Code scanning alerts`を表示します。

2. 結果について判断しましょう：
    1. 報告された問題
    2. 対応するクエリID
    3. `Common Weakness Enumeration` （CWE）識別子
    4. 問題を解決するための推奨事項
    5. `source`から`sink`までのパス。どこに修正を適用しますか。
    6. *真陽性*（*true positive*）ですか*偽陽性*（*false positive*）ですか。

### PRでの結果トリアージ

デフォルトのワークフロー設定では、PRのCode Scanningが有効になっています。
次のステップに従って、実際の動作を確認してください。

1. コードの脆弱なスニペットを追加し、Patch BranchにコミットしてPRを作成します。

    `frontend/src/components/AuthorizationCallback.vue:27` ファイルを変更してください。

    ```javascript
     - if (this.hasCode && this.hasState) {
     + eval(this.code)    
     + if (this.hasCode && this hasState) {
    ```

2. PR で脆弱性が検出されるでしょうか。

3. Code Scanningのチェックの失敗を設定することも可能です。`Code security and analysis`の設定で、`Check Failures`を変更してください。この設定を `Only critical/Only errors`に設定し、その後の PR のチェックでコードスキャンの状態チェックにどのような影響があるか確認してください。次のステップでは、他の重大度タイプを持つ追加のクエリスイートを有効にします。

#### _Stretch Exercise: Fixing false positive results_

偽陽性（False Positive）を確認した場合、どのように対処しますか。これがアプリケーション内でよくあるパターンだとしたらどうしますか。

#### _Stretch Exercise: Enabling code scanning on your own repository_

Dependabot、シークレットスキャン、コードスキャンを有効にする方法を学びました。ご自身のリポジトリで各機能を有効にしてみて、どのような結果が得られるか試してみてください。
