翻訳ページ:
https://google.github.io/ExoPlayer/demo-application.html

## Demo application

> The ExoPlayer demo app serves two primary purposes:
> 
>  - To provide a relatively simple yet fully featured example of ExoPlayer usage. The demo app can be used as a convenient starting point from which to develop your own application.
>  - To make it easy to try ExoPlayer. The demo app can be used to test playback of your own content in addition to the included samples.
> 
> This page describes how to get, compile and run the demo app. It also describes how to use it to play your own media.

ExoPlayerのデモアプリは次の2つの大きな理由のために用意してあります:  

 - シンプルでありながら全てのExoPlayer機能の使い方を例示していること. デモアプリがアプリ開発を始める際の開始点としてとして便利であること.  
 - ExoPlayerをお手軽に試すことができること. デモアプリにオリジナルコンテンツを追加して再生のテストにも使えること.  

このページではどうやってデモアプリを取得し, コンパイルし, 実行するのかを説明します.  
また, あなたが用意したメディアをどうやって再生するのかについても説明します.  

### Getting the code

> The source code for the demo app can be found in the `demo` folder of our [GitHub project](https://github.com/google/ExoPlayer). If you haven’t already done so, clone the project into a local directory:
> 
> `git clone https://github.com/google/ExoPlayer.git`
> 
> Next, open the project in Android Studio. You should see the following in the Android Project view (the relevant folders of the demo app have been expanded):

デモアプリのソースコードはGitHubプロジェクトから取得できます. まだ取得していないのであればローカルディレクトリにcloneしてください.  

  `git clone https://github.com/google/ExoPlayer.git`

次に, AndroidStudioでAndroid Projectビューを開くと次の表示を確認できます.  

![Figure 1. The project in Android Studio](https://google.github.io/ExoPlayer/images/demo-app-project.png)


### Compiling and running

> To compile and run the `demo` app, select and run the demo configuration in Android Studio. The demo app will install and run on a connected Android device. We recommend using a physical device if possible. If you wish to use an emulator instead, please read [FAQ - Does ExoPlayer support emulators?](https://google.github.io/ExoPlayer/faqs.html#does-exoplayer-support-emulators) and ensure that your Virtual Device uses a system image with an API level of at least 23.

AndroidStudioのConfigurationから`demo`を選択し, デモアプリをコンパイル, 実行してください. 接続されているデバイスにデモアプリがインストールされて実行されます. 可能であればエミュレータではなく物理的なデバイスの使用を推奨します. 

もし代わりにエミュレータを使用したいのであれば[FAQ - Does ExoPlayer support emulators?](https://google.github.io/ExoPlayer/faqs.html#does-exoplayer-support-emulators)を読んでください. そうすればAPI lv23以降の仮想デバイスで使うことができます. 

![Figure 2. SampleChooserActivity and PlayerActivity](https://google.github.io/ExoPlayer/images/demo-app-screenshots.png)

> The demo app presents of a list of samples (`SampleChooserActivity`). Selecting a sample will open a second activity (`PlayerActivity`) for playback. The demo features playback controls and track selection functionality. It also has an `EventLogger` class that outputs useful debug information to the system log. This logging can be viewed (along with error level logging for other tags) with the command:

デモアプリの`SampleChooserActivity`にサンプルのリストがあります. リスト項目を選択すればそれを再生するための`PlayerActivity`が起動します. デモは再生コントロールとトラック選択の機能を持っています. また, システムログに有用なデバッグ情報を出力する`EventLogger`クラスを持っています. 次のコマンドで`EventLogger`の有効にして, ほかのログはエラー情報のみ表示することができます. 

`adb logcat EventLogger:V *:E`



### Including extension decoders

> ExoPlayer has a number of extensions that allow use of bundled software decoders, including VP9, Opus, FLAC and FFMPEG (audio only). The demo app can be built to include and use these extensions as follows:
> 
> 1. Build each of the extensions that you want to include. Note that this is a manual process. Refer to the `README.md` file in each extension for instructions.
> 2. In Android Studio’s Build Variants view, change the build variant for the demo module from `demoDebug` to `demo_extDebug`, as shown in Figure 3.
> 3. Compile, install and run the `demo` configuration as normal.

ExoPlayerはいくつかのエクステンションがありVP9, Opus, FLAC and FFMPEG (audio only)といったソフトウェアデコーダを使うことができます. デモアプリでこれらを使うための手順は次の通りです:

 1. 取り込みたいExtensionをビルドします. これは手作業での変更になります. それぞれのExtensionにあるReadmeには手順が示されています. 
 2. Figure 3を参考に, AndroidStudioのビルドバリアントビューでデモモジュールのビルドバリアントを`demoDebug`から`demo_extDebug`に切り替えます. 
 3. いつもの通りデモをコンパイル, インストール, 実行します. 

![](https://google.github.io/ExoPlayer/images/demo-app-build-variants.png)
Figure 3. Selecting the demo_extDebug build variant

> By default an extension decoder will be used only if a suitable platform decoder does not exist. It is possible to indicate that extension decoders should be preferred, as described in the sections below.

デフォルトのデコーダ拡張は適切なプラットフォームデコーダが存在しない時だけ使われる。これはのそセクションで拡張デコーダが望ましいことを示すことができる。


### Playing your own content

> There are multiple ways to play your own content in the demo app.

デモアプリで手持ちのコンテンツを再生するにはいくつかの方法があります.  


#### 1. Editing assets/media.exolist.json

> The samples listed in the demo app are loaded from `assets/media.exolist.json`. By editing this JSON file it’s possible to add and remove samples from the demo app. The schema is as follows, where [O] indicates an optional attribute.

デモアプリでは`asset/media.exolist.json`から読み込まれたサンプルコンテンツがリストされています. このJSONファイルを編集することでこれらのサンプルに追加したり削除したりすることができます. JSONの形式は次の通り. [O]はオプションの属性です.  

```json
[
  {
    "name": "Name of heading",
    "samples": [
      {
        "name": "Name of sample",
        "uri": "The URI/URL of the sample",
        "extension": "[O] Sample type hint. Values: mpd, ism, m3u8",
        "prefer_extension_decoders": "[O] Boolean to prefer extension decoders",
        "drm_scheme": "[O] Drm scheme if protected. Values: widevine, playready",
        "drm_license_url": "[O] URL of the license server if protected",
        "drm_key_request_properties": "[O] Key request headers if protected"
      },
      ...etc
    ]
  },
  ...etc
]
```

> Playlists of samples can be specified using the schema:

サンプルのプレイリストはスキーマを使って特定できる.  

```json
[
  {
    "name": "Name of heading",
    "samples": [
      {
        "name": "Name of playlist sample",
        "prefer_extension_decoders": "[O] Boolean to prefer extension decoders",
        "drm_scheme": "[O] Drm scheme if protected. Values: widevine, playready",
        "drm_license_url": "[O] URL of the license server if protected",
        "drm_key_request_properties": "[O] Key request headers if protected"
        "playlist": [
          {
            "uri": "The URI/URL of the first sample in the playlist",
            "extension": "[O] Sample type hint. Values: mpd, ism, m3u8"
          },
          {
            "uri": "The URI/URL of the first sample in the playlist",
            "extension": "[O] Sample type hint. Values: mpd, ism, m3u8"
          },
          ...etc
        ]
      },
      ...etc
    ]
  },
  ...etc
]
```

> If required, key request headers are specified as an object containing a string attribute for each header:

もし必要なら, それぞれのヘッダに文字列を含むオブジェクトのキーリクエストヘッダを指定できます.  

```json
"drm_key_request_properties": {
  "name1": "value1",
  "name2": "value2",
  ...etc
}
```


#### 2. Loading an external exolist.json file

> The demo app can load external JSON files using the schema above and named according to the `*.exolist.json` convention. For example if you host such a file at `https://yourdomain.com/samples.exolist.json`, you can open it in the demo app using:

デモアプリは上記のスキーマを使った`*.exolist.json`にマッチするJSONを読み込むことができます. もしファイルをhttps://yourdomain.com/samples.exolist.jsonでホストしているならデモアプリを使って次のコマンドで開くことができます. 

`adb shell am start -d https://yourdomain.com/samples.exolist.json`

> Clicking a `*.exolist.json` link (e.g. in the browser or an email client) on a device with the demo app installed will also open it in the demo app. Hence hosting a `*.exolist.json` JSON file provides a simple way of distributing content for others to try in the demo app.

ブラウザやemailクライアントで`*.exolist.json`のリンクをクリックするとデモアプリがインストールされていれば起動することができます. デモアプリで試すには`*.exolist.json`のJSONファイルをホストするのがコンテンツを他へ配信するのに簡単な方法です.  


#### 3. Firing an intent

> Intents can be used to bypass the list of samples and launch directly into playback. To play a single sample set the intent’s action to `com.google.android.exoplayer.demo.action.VIEW` and its data URI to that of the sample to play. Such an intent can be fired from the terminal using:

Intentはサンプルリストをバイパスするのと直接再生することに使えます. 1つのサンプルセットを再生するにはアクション`com.google.android.exoplayer.demo.action.VIEW`とデータのURIが設定されたIntentを使います. そうしたIntentをターミナルから発行するには次のコマンドを使います.  

```bash
adb shell am start -a com.google.android.exoplayer.demo.action.VIEW \
    -d https://yourdomain.com/sample.mp4
```

> Supported optional extras for a single sample intent are:
> 
>  - `extension` [String] Sample type hint. Valid values: mpd, ism, m3u8
>  - `prefer_extension_decoders` [Boolean] Whether extension decoders are preferred to platform ones
>  - `drm_scheme_uuid` [String] Drm scheme UUID if protected
>  - `drm_license_url` [String] Url of the license server if protected
>  - `drm_key_request_properties` [String array] Key request headers packed as name1, value1, name2, value2 etc. if protected

IntentがサポートしているオプショナルなExtraは次の通り: 

 - `extension` [String] サンプルの種類. 使える値は mpd ism m3u8
 - `prefer_extension_decoders` [Boolean] extensionデコーダーがプラットフォームに適しているかどうか
 - `drm_scheme_uuid` [String] 保護されている場合のDrm形式のUUID
 - `drm_license_url` [String] 保護されている場合のライセンスサーバーURL
 - `drm_key_request_properties` [String array] 保護されている場合, name1, value1, name2, value2の形でパッケージされたキーリクエストヘッダ

> When using `adb shell am start` to fire an intent, an optional string extra can be set with `--es` (e.g. `--es extension mpd`). An optional boolean extra can be set with `--ez` (e.g. `--ez prefer_extension_decoders TRUE`). An optional string array extra can be set with `--esa` (e.g. `--esa drm_key_request_properties name1,value1`).

Intentの発行に`adb shell am start`を使うなら, Extraオプションの文字列は`--es` (e.g. `--es extension mpd`)で指定できる. booleanであれば`--ez` (e.g. `--ez prefer_extension_decoders TRUE`). String配列であれば`--esa` (e.g. `--esa drm_key_request_properties name1,value1`)を使って設定することができる.  

> To play a playlist of samples set the intent’s action to `com.google.android.exoplayer.demo.action.VIEW_LIST` and use a `uri_list` string array extra instead of a data URI. For example:

サンプルのプレイリストを再生するにはIntentのアクション`com.google.android.exoplayer.demo.action.VIEW_LIST`と文字配列`uri_list`にデータURIを指定します. 例えば, 

```bash
adb shell am start -a com.google.android.exoplayer.demo.action.VIEW_LIST \
    --esa uri_list https://a.com/sample1.mp4,https://b.com/sample2.mp4
```

> Supported optional extras for a playlist intent are:

プレイリストのIntentでサポートされているオプションは次の通り:  

> - `extension_list` [String array] Sample type hints. Entries may be empty or one of: mpd, ism, m3u8
> - `prefer_extension_decoders`, `drm_scheme_uuid`, `drm_license_url` and `drm_key_request_properties`, all as described above

 - `extension_list` [String array] サンプルタイプのヒント. 内容は空かmpd, ism, m3u8のうちどれかです
 - `prefer_extension_decoders`, `drm_scheme_uuid`, `drm_license_url` と `drm_key_request_properties`それぞれは上記で説明されています. 

