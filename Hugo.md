## Hugo

メソッドのIn, Outとパラメータ, リターン値をログ出力するコードをアスペクトするライブラリ.  
https://github.com/JakeWharton/hugo

ログ出力したいクラス/メソッドに`@DebugLog`でアノテートすることで使用できる.  

```java
// クラス単位での指定が可能
@DebugLog
public class Hobbit {
  ...
}
```

```java
// メソッド単位での指定も可能
@DebugLog
public String getName(String first, String last) {
  SystemClock.sleep(15); // Don't ever really do this!
  return first + " " + last;
}
```

```text
V/Example: ⇢ getName(first="Jake", last="Wharton")
V/Example: ⇠ getName [16ms] = "Jake Wharton"
```


## Hugoの無効化

BuildVariantの`debuggable`が`false`.  

```gralde
release {
    debuggable false
}
```

または, Hugo単体で無効化できる.  

```gradle
hugo {
    enabled
}
```

`@DebugLog`の作用は上記で無効化すればソフトウェアへの影響を0にできる.  
もし実行時にHugoのOn/Offを切り替えるのであれば次の方法が使える.  

```java
Hugo.setEnabled(true|false)
```


Gradle console.  
  Debuggable falseによる無効化  
    "Skipping non-debuggable build type ..."  

  hugo.enabled falseによる無効化  
    "Hugo is not disabled."  


## 導入

```gradle
buildscript {
  repositories {
    mavenCentral()
  }

  dependencies {
    classpath 'com.jakewharton.hugo:hugo-plugin:1.2.1'
  }
}

apply plugin: 'com.android.application'
apply plugin: 'com.jakewharton.hugo'
```


以上です.  
