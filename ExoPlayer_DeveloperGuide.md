
翻訳元ページ：
http://google.github.io/ExoPlayer/guide.html#sending-messages-to-components


## ExoPlayer - Developer guide

> This guide is for ExoPlayer 2.x. If you’re still using 1.x, you can find the old developer guide [here](http://google.github.io/ExoPlayer/guide-v1.html).

本稿はExoPlayer2.x向けの開発者ガイドになります. もしExoPlayer1.xをまだ使用しているのであれば, [古いバージョンの開発者ガイド]((http://google.github.io/ExoPlayer/guide-v1.html))をご覧ください.    

> Playing videos and music is a popular activity on Android devices. The Android framework provides [MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer.html) as a quick solution for playing media with minimal code. It also provides low level media APIs such as [MediaCodec](https://developer.android.com/reference/android/media/MediaCodec.html), [AudioTrack](https://developer.android.com/reference/android/media/AudioTrack.html) and [MediaDrm](https://developer.android.com/reference/android/media/MediaDrm.html), which can be used to build custom media player solutions. 

Android端末で音声と動画を再生することはよくあることです. メディアコンテンツを再生するにはAndroid Frameworkが提供している`MediaPlayer`を使うのが簡単な方法です. また, `MediaCodec`, `AudioTrack`, `MediaDrm`といった低レベルのメディアAPIも提供しており, オリジナルのメディアプレイヤーを実装することもできます.  

> ExoPlayer is an open source, application level media player built on top of Android’s low level media APIs. The open source project contains both the ExoPlayer library and a demo app:
> 
>  - [ExoPlayer library](https://github.com/google/ExoPlayer/tree/release-v2/library) – This part of the project contains the core library classes.
>  - [Demo app](https://github.com/google/ExoPlayer/tree/release-v2/demo) – This part of the project demonstrates usage of ExoPlayer.

ExoPlayerはそうした低レベルのメディアAPIの上に構築されたアプリケーションレベルのメディアプレイヤーでオープンソースプロジェクトとして公開されています. このプロジェクトではExoPlayerライブラリとデモアプリの両方を公開しています.

 - ExoPlayerライブラリ - ExoPlayerのコアライブラリ
 - Demoアプリ - ExoPlayerを使ったデモンストレーション

> This guide describes the ExoPlayer library and its use. It refers to code in the demo app in order to provide concrete examples. The guide touches on the pros and cons of using ExoPlayer. It shows how to use ExoPlayer to play DASH, SmoothStreaming and HLS adaptive streams, as well as formats such as MP4, M4A, FMP4, WebM, MKV, MP3, Ogg, WAV, MPEG-TS, MPEG-PS, FLV and ADTS (AAC). It also discusses ExoPlayer events, messages, customization and DRM support.

この開発者ガイドではExoPlayerライブラリと, その使い方について説明します. DemoアプリではExoPlayerの具体的な使用例を提示します. 
この開発者ガイドではExoPlayerを使用するにあたっての良い点/悪い点について触れ, ExoPlayerでDASH, SmoothStreaming, HLS adaptive streamsやMP4, M4A, FMP4, WebM, MKV, MP3, Ogg, WAV, MPEG-TS, MPEG-PS, FLV and ADTS (AAC)といったフォーマットをどう再生するかについても述べます. また, ExoPlayerのイベント, メッセージ, カスタマイズ, DRMサポートについても議論します.  

### Pros and cons

> ExoPlayer has a number of advantages over Android’s built in MediaPlayer:
> 
> - Support for Dynamic Adaptive Streaming over HTTP (DASH) and SmoothStreaming, neither of which are supported by MediaPlayer. Many other formats are also supported. See the Supported formats page for details.
> - Support for advanced HLS features, such as correct handling of #EXT-X-DISCONTINUITY tags.
> - The ability to seamlessly merge, concatenate and loop media.
> - The ability to customize and extend the player to suit your use case. ExoPlayer is designed specifically with this in mind, and allows many components to be replaced with custom implementations.
> - Easily update the player along with your application. Because ExoPlayer is a library that you include in your application apk, you have control over which version you use and you can easily update to a newer version as part of a regular application update.
> - Fewer device specific issues.
> - Support for Widevine common encryption on Android 4.3 (API level 18) and higher.

ExoPlayerはAndroidにビルトインされているMediaPlayerよりもいくつかの点で優れています.  

 - MediaPlayerではサポートされていないDynamic Adaptive Streaming over HTTP（DASH）とSmoothStreamingをサポートしています. 他にも多くの[フォーマットをサポート](https://google.github.io/ExoPlayer/supported-formats.html)しています.
 - `#EXT-X-DISCONTINUITY`タグなどをハンドルする高度なHLS機能のサポート
 - シームレスなメディアのマージ, 結合, ループ機能
 - ユースケースにあわせてカスタマイズ, 拡張できる機能. ExoPlayerはカスタマイズ性, 拡張性を重視してデザインされており, 多くのコンポーネントがカスタマイズできるようになっています.
 - プレイヤーを簡単にアップデートできます. ExoPlayerはAPKに内包するライブラリ形式であるため, 一般的なアプリケーションのアップデートによって新しいバージョンのプレイヤーに更新することができます. 
 - デバイス固有の問題がより少なくなります
 - Android4.3以降でWidevineの共通暗号化をサポートします

> It’s important to note that there are also some disadvantages:
> 
> - **ExoPlayer’s standard audio and video components rely on Android’s `MediaCodec` API, which was released in Android 4.1 (API level 16). Hence they do not work on earlier versions of Android. Widevine common encryption is available on Android 4.3 (API level 18) and higher.**

重要な注意点として, いくつかのデメリットも挙げておきます.

 - ExoPlayerは標準の音声と動画コンポーネントはAndroidの`MediaCodec`APIに依存しており, それらはAndroid4.1でリリースされたものですので, それより古いAndroid端末では動作しません. さらに, WideVine共通暗号化はAndroid4.3以上で有効になります.  

### Library overview

> At the core of the ExoPlayer library is the `ExoPlayer` interface. An `ExoPlayer` exposes traditional high-level media player functionality such as the ability to buffer media, play, pause and seek. Implementations are designed to make few assumptions about (and hence impose few restrictions on) the type of media being played, how and where it is stored, and how it is rendered. Rather than implementing the loading and rendering of media directly, `ExoPlayer` implementaitons delegate this work to components that are injected when a player is created or when it’s prepared for playback. 

ExoPlayerライブラリのコアは`ExoPlayer`インタフェースです. このインタフェースは, よくあるハイレベルなメディアプレイヤーの機能であるメディアのバッファや再生, 一時停止, シークといった機能を提供します. 実装は再生されているメディアの種類, どうやって/どこにメディアが保存されているか, どうやってレンダリングされているかといったことを推測して（またいくつかの制限の元）デザインされます. メディアを直接読み込んでレンダリングする実装ではなく, `ExoPlayer`の実装はこれをプレイヤーが生成される際や再生時の準備処理のなかでインジェクトされるコンポーネントに任せています. 

> Components common to all `ExoPlayer` implementations are:
> 
> - A `MediaSource` that defines the media to be played, loads the media, and from which the loaded media can be read. A `MediaSource` is injected via `ExoPlayer.prepare` at the start of playback.
> - `Renderers` that render individual components of the media. `Renderers` are injected when the player is created.
> - A `TrackSelector` that selects tracks provided by the `MediaSource` to be consumed by each of the available `Renderers`. A `TrackSelector` is injected when the player is created.
> - A `LoadControl` that controls when the `MediaSource` buffers more media, and how much media is buffered. A `LoadControl` is injected when the player is created.

全てのExoPlayer実装のコンポートネントに共通なことは:

 - `MediaSource`は再生するメディアを定義し, それを読み込み, ロードされたものを読み取ることができます. `MediaSource`は`ExoPlayer.prepare`経由で再生時に設定されます.
 - `Renderers`は個々のメディアコンポーネントをレンダリングします. `Renderers`はプレイヤーが生成される際に設定されます
 - `TrackSelector`は, `MediaSource`によって提供され, 有効な`Renderers`によって消費されるトラックを選択します. `TrackSelector`はプレイヤーが生成される際に設定されます.
 - `LoadControl`は`MediaSource`がよりメディアをバッファする場合や, どれだけメディアをバッファするかを制御します. `LoadControl`はプレイヤーが生成される際に設定されます.

> The library provides default implementations of these components for common use cases, as described in more detail below. An `ExoPlayer` can make use of these components, but may also be built using custom implementations if non-standard behaviors are required. For example a custom `LoadControl` could be injected to change the player’s buffering strategy, or a custom `Renderer` could be injected to use a video codec not supported natively by Android.

ライブラリは, これらのコンポーネントのデフォルト実装を一般ユースケース向けに提供します. `ExoPlayer`はこれらのコンポーネントを利用することができますが, 標準でない振る舞いを必要とされる場合はカスタマイズが必要になります. 例えば, カスタマイズされた`LoadControl`はプレイヤーのバッファリング方法を変更することができます. または, カスタマイズされた`Renderer`はAndroidネイティブでサポートされていないビデオコーデックを使えるようにできます.  

> The concept of injecting components that implement pieces of player functionality is present throughout the library. The default implementations of the components listed above above delegate work to further injected components. This allows many sub-components to be individually replaced with custom implementations. For example the default `MediaSource` implementations require one or more `DataSource` factories to be injected via their constructors. By providing a custom factory it’s possible to load data from a non-standard source or through a different network stack.

コンポーネントのコンセプトはライブラリに存在しているプレイヤーの部分的な実装です. コンポーネントのデフォルトの実装は上記リストで述べた通りインジェクトされたコンポーネントに処理が委譲されます. これで多くのサブコンポーネントで個々のカスタム実装に置き換え可能となります. たとえば`MediaSource`のデフォルト実装は1つかそれ以上の`DataSource`ファクトリをコンストラクタに注入することを要求します. ここにカスタムファクトリを渡せば標準ではないソースや異なるネットワークスタックから読み込むことを可能にします.  


### Getting started

> For simple use cases, getting started with `ExoPlayer` consists of implementing the following steps:
> 
>  1. Add ExoPlayer as a dependency to your project.
>  2. Create a `SimpleExoPlayer` instance.
>  3. Attach the player to a view (for video output and user input).
>  4. Prepare the player with a `MediaSource` to play.
>  5. Release the player when done.
> 
> These steps are outlined in more detail below. For a complete example, refer to `PlayerActivity` in the ExoPlayer demo app.

単純なユースケースで, `ExoPlayer`を構築するには次のステップを踏みます:

 1. ExoPlayerのdependencyをプロジェクトに追加
 2. `SimpleExoPlayer`インスタンスを生成
 3. ビデオ出力とユーザインプットのためにプレイヤーをViewのアタッチ
 4. `MediaSource`を使ってプレイヤーの再生を準備
 5. 再生が終わったらプレイヤーをリリース

これらのステップに関する詳細は次以降に示します. 完全なサンプルはExoPlayerデモアプリにある`PlayerActivity`を参照してください.  

#### Add ExoPlayer as a dependency

> The first step to getting started is to make sure you have the jcenter repository included in the `build.gradle` file in the root of your project.

最初のステップはプロジェクトルートの`build.gradle`にjcenterリポジトリが含まれているかを確認します.  

```gradle
repositories {
    jcenter()
}
```

> Next add a gradle compile dependency for the ExoPlayer library to the build.gradle file of your app module.

次に, アプリモジュールへExoPlayerの依存性を追加します.  

```gradle
compile 'com.google.android.exoplayer:exoplayer:r2.X.X'
```

> where `r2.X.X` is the your preferred version. For the latest version, see the project’s [Releases](https://github.com/google/ExoPlayer/releases). For more details, see the project on [Bintray](https://bintray.com/google/exoplayer/exoplayer).

`r2.X.X`の部分はあなたが期待するバージョンの置き換えて下さい. 最新バージョンはGitHubプロジェクトの[Releases](https://github.com/google/ExoPlayer/releases)を参照してください. より詳細は[Bintray](https://bintray.com/google/exoplayer/exoplayer)をご確認ください.  


#### Creating the player

> Now you can create an `ExoPlayer` instance using `ExoPlayerFactory`. The factory provides a range of methods for creating `ExoPlayer` instances with varying levels of customization. For the vast majority of use cases the default `Renderer` implementations provided by the library are sufficient. For such cases one of the `ExoPlayerFactory.newSimpleInstance` methods should be used. These methods return `SimpleExoPlayer`, which extends ExoPlayer to add additional high level player functionality. The code below is an example of creating a `SimpleExoPlayer`.

これで`ExoPlayerFactory`から`ExoPlayer`インスタンスを得ることができるようになります. ファクトリは多様にカスタマイズされた`ExoPlayer`を生成できるメソッドをもっています. 最もよくあるユースケースはライブラリからデフォルトの`Renderer`実装が十分に提供されている. そういったケースの1つで`ExoPlayerFactory.newSimpleInstance`メソッドは使うべき. これらのメソッドは`SimpleExoPlayer`を返します. これは`ExoPlayer`を拡張しておりプレイヤーのハイレベルな機能として追加されています. 下記のコードは`SimpleExoPlayer`を生成する例です.  

```java
// 1. Create a default TrackSelector
Handler mainHandler = new Handler();
BandwidthMeter bandwidthMeter = new DefaultBandwidthMeter();
TrackSelection.Factory videoTrackSelectionFactory =
    new AdaptiveVideoTrackSelection.Factory(bandwidthMeter);
TrackSelector trackSelector =
    new DefaultTrackSelector(mainHandler, videoTrackSelectionFactory);

// 2. Create a default LoadControl
LoadControl loadControl = new DefaultLoadControl();

// 3. Create the player
SimpleExoPlayer player =
    ExoPlayerFactory.newSimpleInstance(context, trackSelector, loadControl);
```

#### Attaching the player to a view

> The ExoPlayer library provides a `SimpleExoPlayerView`, which encapsulates a `PlaybackControlView` and a `Surface` onto which video is rendered. A `SimpleExoPlayerView` can be included in your application’s layout xml. Binding the player to the view is as simple as:

ExoPlayerライブラリは`SimpleExoPlayerView`を提供し, これはビデオレンダリングの上に`PlaybackControlView`と`Surface`を含んでいます. `SimpleExoPlayerView`はアプリのlayout.xmlに含めることができます. Viewにプレイヤーをバインドするには次のようにシンプルです:  

```java
// Bind the player to the view.
simpleExoPlayerView.setPlayer(player);
```

> If you require fine-grained control over the player controls and the Surface onto which video is rendered, you can set the player’s target `SurfaceView`, `TextureView`, `SurfaceHolder` or `Surface` directly using `SimpleExoPlayer`’s  `setVideoSurfaceView`, `setVideoTextureView`, `setVideoSurfaceHolder` and `setVideoSurface` methods respectively. You can use `PlaybackControlView` as a standalone component, or implement your own playback controls that interact directly with the player. `setTextOutput` and `setId3Output` can be used to receive caption and ID3 metadata output during playback.

もし, プレイヤーのものを超えてきめ細かい制御が必要であったり, ビデオレンダリングされたサーフェースにもそれを求める場合, プレイヤーのターゲットに`SurfaceView`, `TextureView`, `SurfaceHolder`または`Surface`をダイレクトに設定できます. 設定するには`SimpleExoPlayer`の`setVideoSurfaceView`, `setVideoTextureView`, `setVideoSurfaceHolder`, `setVideoSurface`メソッドそれぞれを使うことができます. `PlaybackControlView`を単独のコンポーネント, またはプレイヤーと直接やりとりする独自の再生制御として使うことができます. `setTextOutput`と`setId3Output`は再生中の字幕やID3メタデータ出力に使えます.  

#### Preparing the player

> In ExoPlayer every piece of media is represented by `MediaSource`. To play a piece of media you must first create a corresponding `MediaSource` and then pass this object to `ExoPlayer.prepare`. The ExoPlayer library provides `MediaSource` implementations for DASH (`DashMediaSource`), SmoothStreaming (`SsMediaSource`), HLS (`HlsMediaSource`) and regular media files (`ExtractorMediaSource`). These implementations are described in more detail later in this guide. The following code shows how to prepare the player with a `MediaSource` suitable for playback of an MP4 file.  

ExoPlayerのメディアの全ての部品は`MediaSource`に代表されます. メディアの部品を再生するにはまず最初に対応する`MediaSource`を作成し, そのオブジェクトを`ExoPlayer.prepare`に渡します. ExoPlayerライブラリはDASH (`DashMediaSource`), SmoothStreaming (`SsMediaSource`), HLS (`HlsMediaSource`) と主要なメディアファイル(`ExtractorMediaSource`)の実装`Mediasource`を提供します. これらの実装のより詳細な説明は後程あります. 次のコードはどうやって適切な`MediaSource`でプレイヤーを準備してMP4ファイルを再生するかのコードです.

```java
// Measures bandwidth during playback. Can be null if not required.
DefaultBandwidthMeter bandwidthMeter = new DefaultBandwidthMeter();
// Produces DataSource instances through which media data is loaded.
DataSource.Factory dataSourceFactory = new DefaultDataSourceFactory(this,
    Util.getUserAgent(this, "yourApplicationName"), bandwidthMeter);
// Produces Extractor instances for parsing the media data.
ExtractorsFactory extractorsFactory = new DefaultExtractorsFactory();
// This is the MediaSource representing the media to be played.
MediaSource videoSource = new ExtractorMediaSource(mp4VideoUri,
    dataSourceFactory, extractorsFactory, null, null);
// Prepare the player with the source.
player.prepare(videoSource);
```

> Once the player has been prepared, playback can be controlled by calling methods on the player. For example `setPlayWhenReady` can be used to start and pause playback, and the various `seekTo` methods can be used to seek within the media. If the player was bound to a `SimpleExoPlayerView` or `PlaybackControlView` then user interaction with these components will cause corresponding methods on the player to be invoked.

一度プレイヤーが準備されたら, プレイヤーのメソッドを呼んで再生の制御ができます. 例えば`setPlayWhenReady`は再生と停止, `seekTo`メソッドはメディアのシーク操作に使えます. もしプレイヤーが`SimpleExoPlayerView`か`PlaybackControlView`にバインドされた場合はユーザのインタラクションにより, 操作に対応するコンポーネントのメソッドが呼び出される結果となります.  

#### Releasing the player

> It’s important to release the player when it’s no longer needed, so as to free up limited resources such as video decoders for use by other applications. This can be done by calling `ExoPlayer.release`.

### MediaSource

> In ExoPlayer every piece of media is represented by `MediaSource`. The ExoPlayer library provides `MediaSource` implementations for DASH (`DashMediaSource`), SmoothStreaming (`SsMediaSource`), HLS (`HlsMediaSource`) and regular media files (`ExtractorMediaSource`). Examples of how to instantiate all four can be found in `PlayerActivity` in the ExoPlayer demo app. 

ExoPlayerにあるメディアの全ての部品は`MediaSource`として代表されます. ExoPlayerライブラリはDASH (`DashMediaSource`), SmoothStreaming (`SsMediaSource`), HLS (`HlsMediaSource`) と主要なメディアファイル(`ExtractorMediaSource`)のための`MediaSource`実装を提供します. それぞれ4つのメディアをインスタンス化する例はデモアプリの`PlayActivity`で見ることができます.  

> In addition to the MediaSource implementations described above, the ExoPlayer library also provides `MergingMediaSource`, `LoopingMediaSource` and `ConcatenatingMediaSource`. These `MediaSource` implementations enable more complex playback functionality through composition. Some of the common use cases are described below. Note that although the following examples are described in the context of video playback, they apply equally to audio only playback too, and indeed to the playback of any supported media type(s).

MediaSourceへの追加実装について上記で述べましたが, ExoPlayerライブラリもまた`MergingMediaSource`, `LoopingMediaSource`と`ConcatenatingMediaSource`を提供します. これらの`MediaSource`実装を組み合わせることで, より複雑な再生機能を実現することができます. いくつかの一般的にユースケースは以下で述べます. 注意することとして, 以下のサンプルはビデオ再生時を想定して説明しています. これらはオーディオのみの再生においても同等に使えます. そして当然他のサポートしているメディアタイプでも使えます.  


### Side-loading a subtitle file

> Given a video file and a separate subtitle file, `MergingMediaSource` can be used to merge them into a single source for playback.

ビデオファイルと, 分割されたサブタイトルファイルを渡す, `MergingMediaSource`を使えばそれらを1つのソースとして再生できます.  

```java
MediaSource videoSource = new ExtractorMediaSource(videoUri, ...);
MediaSource subtitleSource = new SingleSampleMediaSource(subtitleUri, ...);
// Plays the video with the sideloaded subtitle.
MergingMediaSource mergedSource =
    new MergingMediaSource(videoSource, subtitleSource);
```

#### Seamlessly looping a video

> A video can be seamlessly looped using a `LoopingMediaSource`. The following example loops a video indefinitely. It’s also possible to specify a finite loop count when creating a `LoopingMediaSource`.

`LoopingMediaSource`で動画をスムーズにループ再生できる. 下記のコードは動画をループし続けるコードです. `LoopingMediaSource`を生成する際にループする回数を指定することもできます.  

```java
MediaSource source = new ExtractorMediaSource(videoUri, ...);
// Loops the video indefinitely.
LoopingMediaSource loopingSource = new LoopingMediaSource(source);
```

#### Seamlessly playing a sequence of videos

> `ConcatenatingMediaSource` enables sequential playback of two or more individual `MediaSources`. The following example plays two videos in sequence. Transitions between sources are seamless. There is no requirement that the sources being concatenated are of the same format (e.g. it’s fine to concatenate a video file containing 480p H264 with one that contains 720p VP9). The sources may even be of different types (e.g. it’s fine to concatenate a video with an audio only stream).

`ConcatenatingMediaSource`は2つかそれ以上の個別の`MediaSources`を連続再生させることができます. 次のサンプルコードは2つのビデオを連続再生するものです. メディア間のつなぎはスムーズです. つなぐメディアフォーマットが同じものである必要はありません（例えば480p H264ビデオファイルと720p VP9とを繋ぐこともできます）. メディアソースは違うタイプにすることもできます（例えばビデオとオーディオストリームの組み合わせ）.  

```java
MediaSource firstSource = new ExtractorMediaSource(firstVideoUri, ...);
MediaSource secondSource = new ExtractorMediaSource(secondVideoUri, ...);
// Plays the first video, then the second video.
ConcatenatingMediaSource concatenatedSource =
    new ConcatenatingMediaSource(firstSource, secondSource);
```


#### Advanced composition

> It’s possible to further combine composite `MediaSources` for more unusual use cases. Given two videos A and B, the following example shows how `LoopingMediaSource` and `ConcatenatingMediaSource` can be used together to loop the sequence (A,A,B) indefinitely.

結合された`MediaSource`は, 他にも珍しいユースケースで使えます. AとB, 2つのビデオがあったとして次のサンプルコードでは`LoopingMediasource`と`ConcatenatingMediaSource`を使ってA, A, Bの順でそれぞれループと連続再生をしています.  

```java
MediaSource firstSource = new ExtractorMediaSource(firstVideoUri, ...);
MediaSource secondSource = new ExtractorMediaSource(secondVideoUri, ...);
// Plays the first video twice.
LoopingMediaSource firstSourceTwice = new LoopingMediaSource(firstSource, 2);
// Plays the first video twice, then the second video.
ConcatenatingMediaSource concatenatedSource =
    new ConcatenatingMediaSource(firstSourceTwice, secondSource);
// Loops the sequence indefinitely.
LoopingMediaSource compositeSource = new LoopingMediaSource(concatenatedSource);
```

> The following example is equivalent, demonstrating that there can be more than one way of achieving the same result.

次のコードは同じ結果となるデモコードです.  

```java
MediaSource firstSource = new ExtractorMediaSource(firstVideoUri, ...);
MediaSource secondSource = new ExtractorMediaSource(secondVideoUri, ...);
// Plays the first video twice, then the second video.
ConcatenatingMediaSource concatenatedSource =
    new ConcatenatingMediaSource(firstSource, firstSource, secondSource);
// Loops the sequence indefinitely.
LoopingMediaSource compositeSource = new LoopingMediaSource(concatenatedSource);
```

> **It is important to avoid using the same `MediaSource` instance multiple times in a composition, unless explicitly allowed according to the documentation. The use of firstSource twice in the example above is one such case, since the Javadoc for `ConcatenatingMediaSource` explicitly states that duplicate entries are allowed. In general, however, the graph of objects formed by a composition should be a tree. Using multiple equivalent `MediaSource` instances in a composition is allowed.**

明示的にドキュメントで許容されている場合を除いて, メディアを集合した場合, 使用した同じ`Mediasource`インスタンスを複数回無効化することは重要です. 最初のソースを2回使う先程のサンプルソースが1つのケースです. `ConcatenatingMediaSource`のjavadocで明示的に重複した呼び出しが許可されていることについて述べられています. 一般的に, しかし, オブジェクトグラフの集合はツリー状であるべきです. 複数の同じ`MediaSource`インスタンスをコンポジションは許可されています.  


### Player events

> During playback, your app can listen for events generated by ExoPlayer that indicate the overall state of the player. These events are useful as triggers for updating the app user interface such as playback controls. Many ExoPlayer components also report their own component specific low level events, which can be useful for performance monitoring.

再生中, アプリはExoPlayerが生成するプレイヤーの全体的なイベントをリスンすることができます. これらのイベントは再生コントロールのようなUIを更新する契機として便利です.  多くのExoPlayerコンポーネントは自身のコンポーネントに関する低レベルなイベントをレポートします. これらもパフォーマンスモニタリングに便利です.  


#### High level events

> ExoPlayer allows instances of `ExoPlayer.EventListener` to be added and removed using its `addListener` and `removeListener` methods. Registered listeners are notified of changes in playback state, as well as when errors occur that cause playback to fail.
> 
> Developers who implement custom playback controls should register a listener and use it to update their controls as the player’s state changes. An app should also show an appropriate error to the user if playback fails.

ExoPlayerは`ExoPlayer.EventListener`インスタンスの追加と削除を`addListener`と`removeListener`メソッドで行うことができます. 登録したリスナは再生状態が変更された場合や, 再生が失敗してエラーが発生した場合に通知を受け取ります. 

カスタム再生コントロールを実装する開発者はリスナを登録してプレイヤーの状態変更によってそのコントロールを更新するべきです. また, アプリは再生時にエラーが発生した場合にそれをユーザへ適切に通知するべきです. 

> When using `SimpleExoPlayer`, additional listeners can be set on the player. In particular `setVideoListener` allows an application to receive events related to video rendering that may be useful for adjusting the UI (e.g. the aspect ratio of the `Surface` onto which video is being rendered). Other listeners can be set to on a `SimpleExoPlayer` to receive debugging information, for example by calling `setVideoDebugListener` and `setAudioDebugListener`.

`SimpleExoPlayer`を使っているとき, プレイヤーへ追加のリスナーを登録できます. 特に`setVideoListener`はアプリがビデオレンダリングに関連するイベントを受信することができます. これはUI調整するのに便利です（例えば, ビデオレンダリングの`Surface`アスペクト比調整）. 例えば`setVideoDebugListener`と`setAudioDebugListener`といった他のリスナも`SimpleExoPlayer`に追加してデバッグ情報を受信したりすることができる.


#### Low level events

> In addition to high level listeners, many of the individual components provided by the ExoPlayer library allow their own event listeners. You are typically required to pass a `Handler` object to such components, which determines the thread on which the listener’s methods are invoked. In most cases, you should use a `Handler` associated with the app’s main thread.

ハイレベルなリスナに加えて, ExoPlayerライブラリからイベントをリスンできる多くの個別コンポーネントが提供されています. 一般的に`Handler`オブジェクトをそういったコンポーネントに渡して, メソッドを呼び出すスレッドを決定することが推奨されています. 多くのケースで, アプリのメインスレッドに紐づけられた`Handler`を使用すべきです.  


### Sending messages to components

> Some ExoPlayer components allow changes in configuration during playback. By convention, you make these changes by passing messages through the `ExoPlayer` to the component, using the `sendMessages` or `blockingSendMessages` methods. This approach ensures both thread safety and that the configuration change is executed in order with any other operations being performed on the player.

いくつかのExoPlayerコンポーネントは再生中に設定を変更することができます. しきたりとして, これらの変更は`sendMessages`か`blockingSendMessages`メソッドを使って`ExoPlayer`を経由したメッセージをコンポーネントに送って変更します. このアプローチは双方のスレッドを安全にかつコンフィグレーションの変更がオペレーションが順番通りに実行されてプレイヤー上で作用します. 


### Customization

> One of the main benefits of ExoPlayer over Android’s `MediaPlayer` is the ability to customize and extend the player to better suit the developer’s use case. The ExoPlayer library is designed specifically with this in mind, defining a number of interfaces and abstract base classes that make it possible for app developers to easily replace the default implementations provided by the library. Here are some use cases for building custom components:

Androidの`MediaPlayer`を超えるExoPlayerのメリットの1つに, プレイヤーを開発者の用途にあわせてカスタマイズし拡張できる機能がある. ExoPlayerライブラリは
