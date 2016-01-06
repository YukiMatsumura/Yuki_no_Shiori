# droidKaigi 2015

2015.4.25
http://droidkaigi.github.io/




## 基調講演 (開発テクニック)
Activity, Fragment, CustomView の使い分け -マッチョなActivityにさよならする方法-
あんざいゆき
AB

Fragmentはkeyevent callbackを持たない.

ninjinkunに日本語訳 square fragment 
the cornere

CustomViewにデータ保持させる.
  BaseSavedState を extends
 
 データとViewのマッピング
   Spinner の selectedPosition と サーバのindexポジション等
   Viewがどこを選択されているかなどなど
   "これが選択されていた場合のデータはこれにする...とか"
   これもCustomViewに

@Relations
@IntDef

Fragmentのはまりどころ.

  - ActivityへのcallbackはonAttachから
  - fragmentからstartActivityつかわない
 

## 開発を効率的に進めるられるまでの道程
@cattaka_net
A
住友

テストケースを開発途中から導入する.

 - 外部依存するところにdi (通信処理, db, preference)
 - 投げっぱなしのスレッドを止める(テストケースが進んで途中で落ちたとき何が原因でNGになったかわからん)

DI library - dagger

疑似通信データはandroidTest配下のassetsフォルダに入れる.

よく使うツール・ライブラリ

Mockito, ライブラリ


## 絶対落ちないアプリの作り方
白山　文彦
A

crashlytics

クラッシュランキング

 - FragmentTransactionの取り扱いミス
 - lifecycleの終わったコントローラへの不正アクセス
 - api連携ミス
 - カメラアプリとの連携みす


---
昼休憩


## 初学者に嬉しいAndroid開発環境
hkusu
開発環境・ツール
B



## 大容量データのダウンロード戦略
@misyobun
メンテナンス
A

200MBクラスのダウンロード

MB tiles

DownloadManager. WiFi Mobileを選択できる. Download中断時のresume機能がある. 
  - アプリ内のUI/UXとしてダウンロード体験を組み込みたい
  - ダウンロード状況や結果を見せたくない
  - ダウンロードの優先順位を独自に考慮したい

独自ダウンローダを作ろう. DownloadManagerを参考に

Download
/Manager - facade
/Provider - 要望管理
 ダウンロードするタスクを管理. ダウンロードのバックエンドはSQLite
 ダウンロード状態の管理.

/Service - 要望制御
 IntentService. 
 Manifestに登録するDownloadServiceは別プロセスで.
 ダウンローダは多くのメモリを消費するためプロセスに割り当てられるメモリを贅沢に使う. 
 Activityたちのプロセスが芯でも大丈夫だし

onTaskRemoveは機種によってはよばれない

/Info - ダウンロードmodel
/thread - ダウンロード実体


Threadを多くしても無駄なパケットヘッダが増えて遅くなる.

ダウンロードしたデータの保存.

ダウンロード途中で失敗. ダウンロードしたサイズを覚えておけばhttp range requestで再開できる. 


## 進化するランタイムART
Android の最新動向
kmt-t
A

ART コンパイラ 5.1.0_r1.

dalvik : バイトコードの実行環境
art では dalvik から完全書き換え. 言語はCからC++11
アーキの変更点: 
  JITコンパイラからAOTに変更. 
  JITでは実行時にコンパイル. AOTではインストール時
  64bit CPI (ARM-v8)
 
performance : 19% 高速化

ART実行ファイル:
OAT... AOTこんぱいらが　出力する実行ファイル. dexをコンパイルしたファイル
  機械語はoatファイルに保存

DEX...OATに埋め込まれる.
  メタデータ(anotationなど)はこちらに含まれる. 

OATファイルについて。
	elf形式. .soはelf形式. 
	oatファイルは共有ライブラリ.

ELFについて
	elf header / program header / segment or section / section header

elfの共有ファイルについてsection
	oat objdump

.oat_pachers ImageSpace headにjava object instance data include.

.text compiled method machiine language .

### ART compile

kinds:
	quick / portable / sea compiler
	portable sea compiler was deleted on master branch

why art is faster?
  - compiler 方式が変更

quick compiler arc
   byte code / mir / 最適化 / lir / 機械語
	LIRは物理CPIアーキテクチャに依存しない(MIPS, ARM, X86 OK)

最適化一覧
  ART 反仮想か. 仮想メソッドが特定のメソッド呼び出しで自明であればそれを反対に具体化する. 




## Bitmapは怖くない。
ハードウェア
wasabeef_jp
B

about bitmapFactory.Options

shadow効果で使用する場合はALPHA_8のconfigが有効.

library Picasso / Glide / Fresco

picasso git...静止画像, glide...アニメする

基本glide, 

blurはsampleingを下げてからかけると高速化される. 

RxJava

## つかえるGradleプロジェクトの作り方
開発環境・ツール
A
zaki50

root
 build.gradle
 project ext { グローバル変数定義 }

module
build.gradle
rootProject.ext.hogehoge で子モジュールから参照できる.

commandlineから実行するのがおすすめ. 

## アプリを公開する前に、最低限知っておきたいセキュリティ事項
メンテナンス
Gaku Taniguchi  taoソフトウェア
A

AnCoLe ... 実習形式で学べるツール

## JellyBean と KitKatで実現するマテリアルデザイン
zaim 

JBとKKで77%のシェア.

StatusBarの色変更 kitkat only.



## 僕らのデータ同期プラクティス
Nkzn
開発環境・ツール
B

agri-note
農業生産者向け管理ツール

つらい
　alarmManagerがときどき消える
　intentServiceをたたきすぎてとまらない
　マルチスレッドでDBごりごり...

Androidのsync
syncAdapterでonPerformSync

web archive かこの履歴を調べる方法

evernote synchronization via EDAM. ninjinkun がやくしてくれるかな

full sync,
  歯抜け無く全て、ネットワークにやさしくない

incremental sync
  クライアントがどこまで持っているかをサーバに教えて差分ダウンロード
  ネットワークに優しい

更新時刻をフィールド二持っておく. 
サーバにはそれ以降に変更されたデータをもらう. 

dirty .... 変更があったよフラグ

clean　受け取ったまま
 / add 新規作成された、まだくらいあんとにしかない 
 / mod  変更した
 / delete　けした

でーたをだうんろーど
初回はfull つぎからはincremental
cleanなものにだけ、さくじょする
cleanなものだけ更新
cleanなものだけ add mod delete
status flagをcleanに

競合問題
evernoteはgit mergeてきな動機




## Android と SELinux
androidsola 
Android の最新動向
B

SELinux - kernelの中に強制アクセス制御が実行される. security policy に問い合わせてrootでも
アクセスさせない. 

type enforcement 

 プロセスがアクセスできるリソースを制限する
 プロセスにはドメイン、リソースにはタイプといったラベルを付与する
 ドメインとタイプに対してどんな操作をするのかを定義する
 プロセスが出来る事を定義する
 allow domain type access


domain transition

親プロセスから子プロセスに遷移するとき異なるドメインを割り当てる仕組み
元の権限を呼び出されない。


RBAC
ドメインを束ねてユーザに対して権限を設定する

aosp のセキュリティポリシーファイルの場所 external/sepolicy


### TEを読む
bootanim.te

SELinuxを無効化する setenforce 0
完全無効には成らないけどアクセス拒否したログがでる.

A
Fireside Chat
mhidaka, あんざいゆき, 長澤 太郎 (@ngsw_taro), ninjinkun, yuki930, Nkzn


Android XRef