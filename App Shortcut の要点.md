## App Shortcut の要点

```
! warning ! 
この投稿はAndroid 7.1 Developer Preview1 - App Shortcutの内容をもとにしています. 
今後のバージョンアップで内容が変更されている可能性にご注意ください.
```

App Shortcutについては下記の投稿もご参考ください.

 - [Android: App Shortcut - 概要](http://yuki312.blogspot.jp/2016/10/android-app-shortcut.html)
 - [Android: App Shortcut - ShortcutManager](http://yuki312.blogspot.jp/2016/10/android-app-shortcut-shortcutmanager.html)

### App Shortcut のバリエーション

App Shortcutには2種類あり, さらにショートカットの状態にも2種類あります. 

_Static Shortcut（Manifest Shortcut）_
リソースファイルで定義される静的なショートカットです. 

_Dynamic Shortcut_
`ShortcutManager`のAPIを使って動的に管理されるショートカットです. 

さらに, ショートカットはランチャーへ"ピン留め"（Pin）できる機能があり, Static/Dynamic Shortcutどちらもピン留めが可能です. 

_Pinned Static Shortcut_
Static Shortcutからランチャーへピン留めされたショートカットです.

_Pinned Dynamic Shortcut_
Dynamic Shortcutからランチャーへピン留めされたショートカットです. 


### App Shortcut のピン留め

ショートカットには更新できるタイプのものと更新できないタイプのものがあります.  
Static Shortcutは不変で, アプリのアップデートでのみ変更できます. これは, ピン留めされたStatic Shortcutも同様です.  

| Action | StaticShortcut | DynamicShortcut | Pinned StaticS. | Pinned DynamicS. |
|--------|----------------|-----------------|-----------------|------------------|
| Update | Ver.UP only | always | Ver.UP only | always |


ピン留めされたショートカットは種別を問わずアプリから削除することができず, ユーザ操作を必要とします.  

| Action | StaticShortcut | DynamicShortcut | Pinned StaticS. | Pinned DynamicS. |
|--------|----------------|-----------------|-----------------|------------------|
| Delete | Ver.UP only | always | x | x |

Static Shortcutはアプリから無効化することができません. ショートカットの定義を削除したバージョンにアップデートされるとStatic Shortcutは無効化されます.  

| Action | StaticShortcut | DynamicShortcut | Pinned StaticS. | Pinned DynamicS. |
|--------|----------------|-----------------|-----------------|------------------|
| Disable | Ver.UP only | always | Ver.UP only | always |

ピン留めされたショートカットはバックアップ/リストアや, ショートカットの定義を削除したアプリへのアップデートでも残る性質があります. ショートカットが有効である限りそれに設定されたIntentが飛んでくることを考慮する必要があります.  

| Action | StaticShortcut | DynamicShortcut | Pinned StaticS. | Pinned DynamicS. |
|--------|----------------|-----------------|-----------------|------------------|
| Remain via Ver.UP | no | yes | yes（disable） | yes |

下表は上記をまとめたものです.  

| Action | StaticShortcut | DynamicShortcut | Pinned StaticS. | Pinned DynamicS. |
|--------|----------------|-----------------|-----------------|------------------|
| Update | Ver.UP only | always | Ver.UP only | always |
| Delete | Ver.UP only | always | x | x |
| Disable | Ver.UP only | always | Ver.UP only | always |
| Remain via Ver.UP | no | yes | yes（disable） | yes |


### ShortcutManager API

Static Shortcutは不変であるため, アプリがショートカットを管理するのは主にDynamic Shortcutになります. ショートカットを追加, 変更, 削除, 無効化, およびそれをサポートするAPIは次の通りです.  

#### [ShortcutManager](https://developer.android.com/reference/android/content/pm/ShortcutManager.html)  

_[disableShortcuts(List<String> shortcutIds, CharSequence disabledMessage)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#disableShortcuts(java.util.List%3Cjava.lang.String%3E, java.lang.CharSequence))_
ピン留めされたショートカットを無効化し, ショートカット選択時のエラーメッセージを指定できます.  

_[getDynamicShortcuts()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getDynamicShortcuts())_
すべてのDynamic Shortcutを取得します.  

_[getManifestShortcuts()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getManifestShortcuts())
すべてのStatic Shortcutを取得します. 

_[getPinnedShortcuts()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getPinnedShortcuts())_
すべてのピン留めされたショートカットを取得します.  

_[removeDynamicShortcuts(List<String> shortcutIds)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#removeDynamicShortcuts(java.util.List%3Cjava.lang.String%3E))_
引数`shortcutIds`のDynamic Shortcutを削除します.  

_[addDynamicShortcuts(List<ShortcutInfo> shortcutInfoList)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E))_
Dynamic Shortcutを追加します. すでに存在しているショートカットと同じIDのものがある場合は更新されます. このAPIはRate-limitの対象です.  

_[setDynamicShortcuts(List<ShortcutInfo> shortcutInfoList)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E))_
Dynamic Shortcutを登録します. すでに存在しているショートカットと同じIDのものがある場合は置き換えられます.  ピン留めされたショートカットは不変でないものだけが置き換えられます. このAPIはRate-limitの対象です.  

_[updateShortcuts(List<ShortcutInfo> shortcutInfoList)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E))_
すでに存在するショートカットと同じIDのものを更新します. ピン留めされたショートカットは不変でないものだけが更新できます. このAPIはRate-limitの対象です.  


#### [ShortcutInfo](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html)

_[getExtras()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#getExtras())_
当該ショートカットに設定された, 任意の情報を格納できる`extra`を取得します.  

_[isDeclaredInManifest()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#isDeclaredInManifest())_
当該ショートカットが, マニフェストで定義されるStatic Shortcutであるか否かを取得できます.  

_[isDynamic()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#isDynamic())_
当該ショートカットが, Dynamic Shortcutであるか否かを取得できます.  

_[isEnabled()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#isEnabled())_
当該ショートカットが, 有効な状態であるか否かを取得できます.  

_[isImmutable()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#isImmutable())_
当該ショートカットが, 変更できない不変なものであるか否かを取得できます.  

_[isPinned()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#isPinned())_
当該ショートカットが, ピン留めされたショートカットであるか否かを取得できます.  


### Shortcut Listの並び順

ショートカットリストはランチャーアイコンに近い方から優先度（`rank`）の昇順で並べます（これはランチャーアプリの実装に依存します）. 例えばNexus Launcherであればリストの並ぶ方向はランチャーアイコンの位置によって変わります. 

<a href="https://1.bp.blogspot.com/-11zstE-oWUE/WAtYa1OYPvI/AAAAAAAAN_A/cPJcEb6o11AZ14kimJJDrS69cekaf4ivQCPcB/s1600/Screenshot_1477138077.png" imageanchor="1" ><img border="0" src="https://1.bp.blogspot.com/-11zstE-oWUE/WAtYa1OYPvI/AAAAAAAAN_A/cPJcEb6o11AZ14kimJJDrS69cekaf4ivQCPcB/s400/Screenshot_1477138077.png" width="225" height="400" /></a> <a href="https://1.bp.blogspot.com/-QOsGKxCFRFY/WAtYa8slIRI/AAAAAAAAN-8/EHQaY2fCzjYEYV-r-9lMJSwYUysjpoSjQCPcB/s1600/Screenshot_1477138088.png" imageanchor="1" ><img border="0" src="https://1.bp.blogspot.com/-QOsGKxCFRFY/WAtYa8slIRI/AAAAAAAAN-8/EHQaY2fCzjYEYV-r-9lMJSwYUysjpoSjQCPcB/s400/Screenshot_1477138088.png" width="225" height="400" /></a>

### Shortcutの期限と再利用

ショートカットはユーザに特定のアクションへ素早くアクセスさせるための手段です. それぞれのショートカットはそれを特定するために一意で不変なIDを割り当てられます. このIDはショートカットを登録するアプリが任意に指定できますが, 次の点を考慮し, これを厳守する必要があります. 

 - ピン留めされたショートカットに"残る性質"がある
 - バックアップ/リストアで他端末に復元される可能性
 - 古いバージョンで登録した既にサポートしないショートカットが新しいバージョンのアプリに対して実行される

#### ショートカットに向かないパラメータ

ショートカットはピン留めされると"残ります". そして, 異なる端末にリストアされる可能性もあります. 
例えば, Registration IDのような期限付きの情報をショートカットに紐付けるなどした場合, それが期限切れとなっている可能性を考慮する必要があります.  
また, 端末固有の情報（IMEIなど）は異なる端末にリストアされた場合に間違った情報となることに注意が必要です.  

ショートカットに登録しておく情報は恒久的に変わらない類のものであれば問題ないのですが, そうでない場合はショートカットの情報が古くなったケースや, 端末の状況が変わりショートカットの情報が正しいものではなくなるケースを考慮したコードを用意しておく必要があります.  
そういったコードは, 一般的に起動されるアクティビティの`onCreate`に記述します.  


#### Dynamic Shortcut IDを無理やりStatic Shortcut IDに

Dynamic Shortcutで使用されているIDを, アプリバージョンアップを経てStatic ShortcutのIDで再利用した場合の仕様は現状記載がありません.  
Android7.1 emulatorのNexus Launcherアプリで確認する限り, すでに登録されているDynamic Shortcut IDをStatic Shortcutで上書きしようとしても失敗し, Dynamic/Static Shortcutは両方存在しなくなりました. 
事前にDynamic Shortcutをピン留めしていた場合, これはDynamic Shortcutのピン留めショートカットとして振る舞うため無効化が可能な状態になります.  

このあたりの動作はLauncherアプリの実装に依存するところもあるでしょうし, そもそもShortcut IDの一意性や永続性を厳守すれば考えなくてもよい問題です.  
Shortcut IDは一意に, 意味が変わるようであれば新規IDを払い出し, IDの再利用は避けるのが吉です.  


### Launcher Applicationの実装

Android7.1の端末であればApp Shortcut機能を持ったランチャーアプリが必ず搭載されているわけではありません. App Shortcutの機能をサポートするかどうか, ピン留めをサポートするか, App Shortcutをどのようなジェスチャーを契機に表示するかなど, そのほとんどがランチャーアプリに依存します. ショートカットのShort labelも必ず表示されるとは限りません.  


### Shortcutのセキュリティ

他のアプリはあなたのショートカットメタデータにアクセスすることができませんが, ランチャーアプリに限ってはこれにアクセスすることが可能です. そのため, このメタデータにユーザ情報などのセンシティブなデータを含めないようにすべきです.


以上です.  
