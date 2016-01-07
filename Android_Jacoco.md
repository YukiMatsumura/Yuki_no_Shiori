```gradle
apply plugin: 'jacoco'

android {
  jacoco {
    version = '0.7.5.201505241946'
  }

task jacocoTestReport(type: JacocoReport, dependsOn: "connectedCheck") {
  group = "Reporting"
  description = "Generate Jacoco coverage reports"

  reports {
      xml.enabled = false
      html.enabled = true
      html {
          destination "./build/reports/jacoco/${project.name}"
      }
  }

  classDirectories = fileTree(
      dir: './build/intermediates/classes/development/debug',
      excludes: ['**/R.class',
        '**/R$*.class',
        '**/BuildConfig.*',
        '**/Manifest*.*',

        // ButterKnife Gen.
        '**/*$$ViewBinder.*'
      ]
  )

  sourceDirectories = files('src/main/java')
  executionData = files('./build/jacoco/testDevelopmentDebugUnitTest.exec')
}
```

実行する場合は`jacocoTestReport`を実行する.  

```bash
$ ./gradlew :app:jacocoTestReport
```

excludesの指定について, ButterKnifeライブラリを使用している場合は自動生成される`ViewBinger`クラスを除外する必要がある.  
Jacocoは`$`記号が連続するクラス名を持つ場合にうまく動作しない. ButterKnifeは`$$`を含むクラスを生成するため, これを除外しておく必要がある. 

[参考: Issue 69174:	Gradle Build System Jacoco VerifyException when used with Dagger](https://code.google.com/p/android/issues/detail?id=69174)