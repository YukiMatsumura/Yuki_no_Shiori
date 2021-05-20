ライブラリがパーミッションを定義していると, アプリのパーミッションとして自動で追加される.
Android Studioで`AndroidManifest.xml`を開いて `Merged Manifest`タブを開けば最終的にアプリが使用するパーミッションを確認できる.

ここで, `<uses-permission>` として定義されたパーミッションをどのライブラリが追加・定義しているのかを調べたい場合, アプリを一度ビルドして`[module]/build/outputs/logs/manifest-merger-[build variant]-report.txt`の内容を確認すれば良い.


下記のような出力結果が得られるので, 「`READ_EXTERNAL_STORAGE` は `LeakCanary` が追加しているんだな」と知ることができる.

```
uses-permission#android.permission.READ_EXTERNAL_STORAGE
ADDED from [com.squareup.leakcanary:leakcanary-android-core:2.4] xxxleakcanary-android-core-2.4/AndroidManifest.xml:23:5-80
```

以上.
