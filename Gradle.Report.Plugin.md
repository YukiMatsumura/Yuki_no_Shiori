## Gradle Project Report

Gradle標準にビルド時のプロジェクト状態をレポートする[Project Reports Plugin](https://docs.gradle.org/current/userguide/project_reports_plugin.html)があります.  
本稿はこのプラグインを使ってみたレポートです.  

公式ドキュメントにある[Obtaining information about your build](https://docs.gradle.org/current/userguide/tutorial_gradle_command_line.html#sec:obtaining_information_about_your_build)で触れられているいくつかのコマンドをGradleのタスクとして定義できます.  
導入は非常に簡単で `apply plugin: 'project-report'` を宣言するだけです.  

これにより追加されるタスクについて紹介します.  


### [DependencyReport](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.diagnostics.DependencyReportTask.html)

プロジェクトの依存関係ツリーをテキストファイル形式でレポートするタスクです.  
レポートは`build/reports/project`フォルダ配下に出力されます.  

このタスクはコマンドラインで下記を実行した結果と等価です.  

```gradle
./gradlew dependencies
```

下記のような結果が得られます.  

```text
------------------------------------------------------------
Project :app
------------------------------------------------------------

_debugAndroidTestApk - ## Internal use, do not manually configure ##
No dependencies

_debugAndroidTestCompile - ## Internal use, do not manually configure ##
No dependencies

_debugApk - ## Internal use, do not manually configure ##
+--- com.android.support:appcompat-v7:23.1.1
|    \--- com.android.support:support-v4:23.1.1
|         \--- com.android.support:support-annotations:23.1.1
+--- com.android.support:design:23.1.1
|    +--- com.android.support:appcompat-v7:23.1.1 (*)
|    +--- com.android.support:recyclerview-v7:23.1.1
```

Androidでは依存性の推移で問題が発生する機会が多く, これを解決するためによく使われるコマンドです.  

参考: [気をつけたいGradleの推移的依存関係とその解決 - Qiita](http://qiita.com/tomoima525/items/6dd59830aa122e39520b)  


### [HtmlDependencyReport](https://docs.gradle.org/current/dsl/org.gradle.api.reporting.dependencies.HtmlDependencyReportTask.html)

プロジェクトの依存関係ツリーをHTML形式でレポートするタスクです.  
レポートは`build/reports/project/dependencies`フォルダ配下に出力されます.  

（このレポート形式が見やすいのかは別として...）  
デフォルト設定だと, 各モジュール毎にHTMLレポートの出力を行う必要があります.  
もし, 複数のモジュールを扱うプロジェクトであれば, ルートの`build.gradle`に下記を定義することでプロジェクト全体を対象としたレポートを出力できます.  

```gradle
apply plugin: 'project-report'
htmlDependencyReport {
    projects = project.allprojects
}
```


### [PropertyReport](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.diagnostics.PropertyReportTask.html)

プロジェクトのプロパティをテキストファイル形式でレポートするタスクです.  
レポートは`build/reports/project`フォルダ配下に出力されます.  

このタスクはコマンドラインで下記を実行した結果と等価です.  

```gradle
./gradlew properties
```

下記のような結果が得られます.  

```text
------------------------------------------------------------
Project :app
------------------------------------------------------------

allprojects: [project ':app']
android: com.android.build.gradle.AppExtension_Decorated@1526742
android.injected.invoked.from.ide: true
androidDependencies: task ':app:androidDependencies'
ant: org.gradle.api.internal.project.DefaultAntBuilder@1ef6195
...
```


### [TaskReport](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.diagnostics.TaskReportTask.html)

プロジェクトのタスク一覧をテキストファイル形式でレポートするタスクです.  
レポートは`build/reports/project`フォルダ配下に出力されます.  

このタスクはコマンドラインで下記を実行した結果と等価です.  

```gradle
./gradlew tasks
```

下記のような出力結果が得られます.  

```text
------------------------------------------------------------
All tasks runnable from project :app
------------------------------------------------------------

Android tasks
-------------
androidDependencies - Displays the Android dependencies of the project.
signingReport - Displays the signing info for each variant.
sourceSets - Prints out all the source sets defined in this project.

Build tasks
-----------
assemble - Assembles all variants of all applications and secondary packages.
assembleAndroidTest - Assembles all the Test applications.
assembleDebug - Assembles all Debug builds.
...
```


### [ProjectReport](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.ProjectReportsPluginConvention.html)

ここまで紹介した`dependencyReport`, `propertyReport`, `taskReport`, `htmlDependencyReport`を全て実行するショートカットタスクです.  

各レポートタスクの共通設定が可能です.  

[projectReportDir](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.ProjectReportsPluginConvention.html#org.gradle.api.plugins.ProjectReportsPluginConvention:projectReportDir)
: プロジェクトレポートの出力場所を指定する. 

[projectReportDirName](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.ProjectReportsPluginConvention.html#org.gradle.api.plugins.ProjectReportsPluginConvention:projectReportDirName)
: プロジェクトレポートの出力先フォルダ名を指定する. 

```gradle
projectReport {
    projectReportDirName = "project_reports"
}
```

以上です. 