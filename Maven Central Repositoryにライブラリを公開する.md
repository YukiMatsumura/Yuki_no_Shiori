
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
| Project URL | プロジェクトページのURL. 例：https://github.com/YukiMatsumura/koma |
| SCM url | GitのURLなど. 例：https://github.com/YukiMatsumura/koma.git |
| Username | 空 |
| Already Synced to Central | No |

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
ここで指定するGroupIdに紐づくドメインを所有または管理している必要があります。Issueで申請後、ドメインの所有/管理していることの証明を求められます。
ただし、GitHubやGitLabなど特定のコードホスティングサービスであればドメインの所有権がなくても、個人アカウントレベルのドメインをサポートしています。
https://central.sonatype.org/publish/requirements/coordinates/#supported-code-hosting-services-for-personal-groupid
例えば `github.com/yourusername` のアカウントであれば `io.github.yourusername` をGroup Idとして登録できます。

*GitHubなどコードホスティングサービスの個人ページをGroup Idに指定した場合*
指定のGroup Idがあなたの管理下にあることを証明する必要があります。
作成したIssueのチケット名で空のリポジトリを作成し、アカウントの所有権を証明しましょう。
例：`io.github.myusername`をGroupIdに指定し管理している場合、チケット名`OSSRH-*****`を名前にしたリポジトリ`github.com/myusername/OSSRH-*****`を作成します。

起票したIssueに最長でも2営業日以内に管理者からコメントで返信があるはずです。
反応があるまで待ちましょう。

ドメインの所有権確認などが済めば、リポジトリマネージャが利用できるようになります。
リポジトリマネージャにはJIRAの登録アカウントでログインできます。
https://oss.sonatype.org/

### 2. GPG

Maven Central Repositoryに登録するaarなどのアーティファクトにはGnuPGなどによる署名が必要です。
下記の手順に従ってGPGを導入しましょう。
https://central.sonatype.org/publish/requirements/gpg/

ざっくり手順を書いておきます。

1. インストール

```bash
$ brew install gnupg
```

2. バージョン確認

```bash
$ gpg --version
gpg (GnuPG) 2.2.29
```

3. 鍵生成

```bash
$ LANG=C gpg --full-gen-key
```

 - Kind of key: `1` RSA and RSA.
 - Key size: `4096` 鍵のサイズ.
 - Expiration: `0` 0で無期限. 期限ありにしたいならそれを指定.
 - Real name, email: ご自由に
 - Comment: フリーテキスト. 空でもok.

実行を終えるとキーを保護するためのパスワードを求められるので入力する。

4. 生成した鍵IDを確認

```bash
$ gpg --list-keys
/Users/xxx/.gnupg/pubring.kbx
---------------------------------
pub   rsa2048 2021-xx-xx [SC]
      ABCDEFG0123456789ABCDEFG0123456789ABCDEF
uid           [ultimate] MatsumuraYuki <xxxx@xxx.xxx>
sub   rsa2048 2021-xx-xx [E]
```

これで生成した公開鍵の情報が得られます。
`pub`にあるフィンガープリントの下8桁が鍵IDになります。（ここでは `89ABCDEF`）
この8桁の鍵IDはあとで使うのでメモしておきます。

5. 公開鍵を鍵サーバへ登録

公開鍵があなたのものであることを確認できるように、鍵サーバーにアップロードします。

```bash
$ gpg --keyserver keyserver.ubuntu.com --send-keys <先ほど生成した8桁の鍵ID>
```

現在Maven Central Repositoryがサポートしている鍵サーバは下記の３つです.

 - `keyserver.ubuntu.com`
 - `keys.openpgp.org`
 - `pgp.mit.edu`

6. 秘密鍵のBase64エクスポート

署名する際に使う秘密鍵の情報をBase64エクスポートしてメモしておきます。

```bash
$ gpg --export-secret-keys 5BEF072A | base64
```


### 3. Setup Gradle

ここから先は下記のプロジェクトを参考に進めてみてください。動いている完成形で、これをベースに話を進めます。
https://github.com/YukiMatsumura/koma

公開に必要な設定はルートやモジュールの`build.gradle`とは別ファイルで管理するようにします（必須ではないですが、管理しやすくなるのでファイルを分けます）
プロジェクトルートに `scripts` ディクトリを作成して、そこに `publish-module.gradle` と `publish-root.gradle` の空ファイルを作成しておきます。

#### Root `build.gradle`

次にプロジェクトルートの `build.gradle` に下記を追加します。

```gradle
buildscript {
  repositories {
    maven { url "https://plugins.gradle.org/m2/" }
    ...
  }
  dependencies {
      ...
      classpath 'io.github.gradle-nexus:publish-plugin:1.1.0'
      classpath "org.jetbrains.dokka:dokka-gradle-plugin:1.5.0"
  }
}

apply plugin: 'io.github.gradle-nexus.publish-plugin'
apply from: "${rootDir}/scripts/publish-root.gradle"
```

Maven Central Repositoryへの公開には [gradle-nexus/publish-plugin](https://github.com/gradle-nexus/publish-plugin/)を使います。
また、参考プロジェクトはdokkaを使っているのでそのクラスパスも追加しています。

#### publish-root.gradle

`publish-root.gradle` の内容は次のとおりです。

```gradle
ext["ossrhUsername"] = ''
ext["ossrhPassword"] = ''
ext["sonatypeStagingProfileId"] = ''
ext["signing.keyId"] = ''
ext["signing.password"] = ''
ext["signing.secretKeyRingFile"] = ''

// CIとローカルビルド両方で動作するように秘匿情報の参照先を分けます
File secretPropsFile = project.rootProject.file('local.properties')
if (secretPropsFile.exists()) {
  Properties p = new Properties()
  new FileInputStream(secretPropsFile).withCloseable { is -> p.load(is) }
  p.each { name, value -> ext[name] = value }
} else {
  ext["ossrhUsername"] = System.getenv('OSSRH_USERNAME')
  ext["ossrhPassword"] = System.getenv('OSSRH_PASSWORD')
  ext["sonatypeStagingProfileId"] = System.getenv('SONATYPE_STAGING_PROFILE_ID')
  ext["signing.keyId"] = System.getenv('SIGNING_KEY_ID')
  ext["signing.password"] = System.getenv('SIGNING_PASSWORD')
  ext["signing.key"] = System.getenv('SIGNING_KEY')
}

nexusPublishing {
  repositories {
    sonatype {
      stagingProfileId = sonatypeStagingProfileId
      username = ossrhUsername
      password = ossrhPassword
      // 2021.02以降Maven Central Repositoryにリポジトリを新規作成する場合は下記の指定が必要です
      // https://central.sonatype.org/publish/publish-gradle/#metadata-definition-and-upload
      nexusUrl.set(uri("https://s01.oss.sonatype.org/service/local/"))
      snapshotRepositoryUrl.set(uri("https://s01.oss.sonatype.org/content/repositories/snapshots/"))
    }
  }
}
```


#### Module `build.gradle`

公開するライブラリモジュールの `build.gradle` に下記を追加します。

```gradle
ext {
  // Provide your own coordinates here
  PUBLISH_GROUP_ID = 'Group ID. 例：io.github.yukimatsumura'
  PUBLISH_VERSION = 'ライブラリバージョン. 例：0.2'
  PUBLISH_ARTIFACT_ID = 'アーティファクトID. 例：koma'
}

apply from: "${rootProject.projectDir}/scripts/publish-module.gradle"
```

アーティファクトIDは `implementation "GroupID:ArtifactID:version"` で指定するアーティファクトIDになります。

#### publish-module.gradle

`publish-module.gradle` の内容は次のとおりです。

```gradle
apply plugin: 'maven-publish'
apply plugin: 'signing'
apply plugin: 'org.jetbrains.dokka'

task androidSourcesJar(type: Jar) {
  archiveClassifier.set('sources')
  from android.sourceSets.main.java.srcDirs
  from android.sourceSets.main.kotlin.srcDirs
}

tasks.dokkaHtml.configure {
  outputDirectory.set(file("../documentation/html"))
}

tasks.withType(dokkaHtml.getClass()).configureEach {
  pluginsMapConfiguration.
      set(["org.jetbrains.dokka.base.DokkaBase": """{ "separateInheritedMembers": true}"""])
}

task javadocJar(type: Jar, dependsOn: dokkaJavadoc) {
  archiveClassifier.set('javadoc')
  from dokkaJavadoc.outputDirectory
}

artifacts {
  archives androidSourcesJar
  archives javadocJar
}

signing {
  useInMemoryPgpKeys(rootProject.ext["signing.keyId"],
      rootProject.ext["signing.key"],
      rootProject.ext["signing.password"],)
  sign publishing.publications
}

group = PUBLISH_GROUP_ID
version = PUBLISH_VERSION

afterEvaluate {
  publishing {
    publications {
      release(MavenPublication) {
        groupId PUBLISH_GROUP_ID
        artifactId PUBLISH_ARTIFACT_ID
        version PUBLISH_VERSION

        // Two artifacts, the `aar` (or `jar`) and the sources
        if (project.plugins.findPlugin("com.android.library")) {
          from components.release
        } else {
          from components.java
        }

        artifact androidSourcesJar
        artifact javadocJar

        pom {
          name = PUBLISH_ARTIFACT_ID
          description = 'プロジェクトの概要'
          url = 'プロジェクトのURL. 例：https://github.com/YukiMatsumura/koma'

          licenses {
            license {
              // ライセンス情報
              name = 'The Apache License, Version 2.0'
              url = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
            }
          }
          developers {
            developer {
              id = 'よしなに. 例：YukiMatsumura'
              name = 'よしなに. 例：Matsumura Yuki'
              email = 'よしなに. 例：xxxx@gmail.com'
            }
          }
          scm {
            connection = 'VCS情報. 例：scm:git:github.com/YukiMatsumura/koma.git'
            developerConnection = 'VCS情報. 例：scm:git:ssh://github.com/YukiMatsumura/koma.git'
            url = 'VCS情報. 例：https://github.com/YukiMatsumura/koma/tree/main'
          }
        }
      }
    }
  }
}
```

### 4. local.properties

外部公開できない秘匿情報を`local.properties`に定義しましょう。

```
signing.keyId=公開鍵の8桁ID. 例：89ABCDEF
signing.password=PGPで生成した秘密鍵Base64情報. 例：PMxxxxxxxxxxxxxxxx.........xx==

ossrhUsername=リポジトリマネージャログインID
ossrhPassword=リポジトリマネージャログインパスワード
sonatypeStagingProfileId=ステージングプロファイルID
```

#### ossrhUsername/password

そのままsonatypeのusername/passwordを指定することもできますが、よりセキュアにアクセストークンを発行して指定することもできます。
Sonatypeのリポジトリマネージャで、 `画面右上のログイン名 → Profile → User Token` からトークンを生成し、username/passwordと差し替えます。



#### Staging profile id

https://s01.oss.sonatype.org/ にログイン後, `Build Promotion → Staging Profiles` を選択し, 自分のプロファイルを選択するとURLの末尾にプロファイルIDが表示されます。
これを`sonatypeStagingProfileId`に指定します。

```
例：https://s01.oss.sonatype.org/#stagingProfiles;<profile id>
```

### 5. Release

これですべての設定は完了しました。
Gradleのタスクリストを見ると、ライブラリモジュールのタスクに`publishReleasePublicationToSonatypeRepository`がいるはずです。
コマンドを実行してライブラリをプレリリースしましょう。

```gradhe
./gradlew :<モジュール名>:publishReleasePublicationToSonatypeRepository
```

コマンドを実行すると、Sonatypeリポジトリマネージャの `Build Promotion → Staging Repositories` にライブラリがアップロードされているのがわかります。
そのライブラリを選択し `Close` アクションを実行しましょう。
`Close`を実行するとしばらくの間バリデーションが実行されます。実行状況は同画面の `Activity` タブから確認できます。

リポジトリを閉じると`Drop`か`Release`のアクションが選択可能になります。
公開プロセスで問題があった場合は`Drop`でキャンセルできます。
`Release`を選択するとMaven Centralに公開します。`Release`後はステージングのアイテムは不要なのでDropできます。

公開には10~15分、長いと1時間以上かかります。
正常に公開されると https://repo1.maven.org/maven2/ であなたのリポジトリが参照できます。
さらに数時間後には https://search.maven.org/ で検索が可能になっているはずです。

以上です。
