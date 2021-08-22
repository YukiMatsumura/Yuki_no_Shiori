
手順はここに書いてます。
[The Central Repository Documentation - Getting started](https://central.sonatype.org/publish/publish-guide/)

本稿では必要な手順を端的に書いていきます。

### 1. Create your JIRA account ＆ Issue

Maven Central Repositoryにあなたのレポジトリを作成するには申請する必要があります。
申請はJIRAチケットで行われますので、下記からアカウントを作ります。
https://issues.sonatype.org/secure/Signup!default.jspa

アカウントを作ったら下記リンクからリポジトリ作成のIssueを作ります。
https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134

| フィールド | 値 |
|-|-|
| 要約 | プロジェクト名など　例：YukiMatsumura / koma |
| 説明 | READMEなどプロジェクトの概要　|
| Group Id | あなたのプロジェクトであることを示す識別子. *後述 |

#### Group Id

まずはここを読んだようがいいです。
https://central.sonatype.org/publish/requirements/coordinates/

Group Idはよくある`implementation`指定で使われるもので, 下記でいうと `io.github.yukimatsumura` がGroup Idになります。

```
implementation 'io.github.yukimatsumura:koma:0.2'
```

あなたが今後Maven Central RepositoryにリリースするであろうすべてのプロジェクトがこのGroup Idに紐づきます。
例えば、`example.com`を管理している場合、`com.example.domain`、 `com.example.testsupport`など、`com.example`で始まるGroup Idを使用することができます。

*注意*

起票したIssueに最長でも2営業日以内に管理者からコメントで返信があるはずです。
反応があるまで待ちましょう。
