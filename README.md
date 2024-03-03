
<h1 align="center">GitHub Advanced Security Bootcamp</h1>

> このブートキャンプは、GitHub Advanced Security (GHAS) に慣れ親しみ、ご自身のリポジトリで GitHub Advanced Security (GHAS) を使う方法をより理解できるようにするためのものです。

```bash
git clone https://github.com/ghas-bootcamp/ghas-bootcamp.git
cd ghas-bootcamp
git remote set-url origin git@github.com:{org-or-username}/{repo-name}.git
```

## :octocat: Agenda

### Day 1

- [ ] GitHub Advanced Security とは
- [ ] GitHub Advanced Security のライセンス
- [ ] GHAS の有効化
- [ ] How does it work?

ハンズオン演習のインストラクション: [Developer Exercises - Day 1](exercises/developer-exercises-day1.md)

### Day 2

- [ ] How does it work?(続き)
- [ ] アクセス、通知、アラート
- [ ] 統合 - GHAS APIとWebhooks
- [ ] GHAS のトラブルシューティング

ハンズオン演習のインストラクション: [Developer Exercises - Day 2](exercises/developer-exercises-day2.md)

## :books: Resources
- [About code scanning](https://docs.github.com/ja/github/finding-security-vulnerabilities-and-errors-in-your-code/about-code-scanning)
- [About Dependabot Alerts](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/about-alerts-for-vulnerable-dependencies)
- [About secret scanning](https://docs.github.com/ja/github/administering-a-repository/about-secret-scanning)
- [Events that trigger workflows](https://docs.github.com/ja/free-pro-team@latest/actions/reference/events-that-trigger-workflows)
- [Configuring the CodeQL workflow for compiled languages](
https://docs.github.com/ja/free-pro-team@latest/github/finding-security-vulnerabilities-and-errors-in-your-code/configuring-the-codeql-workflow-for-compiled-languages)
- [Configuring code scanning](https://docs.github.com/ja/free-pro-team@latest/github/finding-security-vulnerabilities-and-errors-in-your-code/configuring-code-scanning)
- [Configuring notifications for Dependabot alerts](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/configuring-notifications-for-vulnerable-dependencies#configuring-notifications-for-dependabot-alerts)
- [Customizing dependency updates](https://docs.github.com/ja/free-pro-team@latest/github/administering-a-repository/customizing-dependency-updates)
- [Configuration options for the dependabot.yml file](https://docs.github.com/ja/free-pro-team@latest/github/administering-a-repository/configuration-options-for-dependency-updates)
- [Filter pattern cheat sheet](https://docs.github.com/ja/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions#filter-pattern-cheat-sheet)
- [Running additional queries](
https://docs.github.com/ja/free-pro-team@latest/github/finding-security-vulnerabilities-and-errors-in-your-code/configuring-code-scanning#running-additional-queries)
- [Troubleshooting the CodeQL workflow](https://docs.github.com/en/free-pro-team@latest/github/finding-security-vulnerabilities-and-errors-in-your-code/troubleshooting-the-codeql-workflow)
- [Code scanning API](https://docs.github.com/ja/free-pro-team@latest/rest/reference/code-scanning)
- [Secret scanning API](https://docs.github.com/ja/rest/reference/secret-scanning)
- [GraphQL API](https://docs.github.com/ja/free-pro-team@latest/graphql)
  - [RepositoryVulnerabilityAlert](https://docs.github.com/ja/free-pro-team@latest/graphql/reference/objects#repositoryvulnerabilityalert)
- [REST API](https://docs.github.com/ja/free-pro-team@latest/rest)
