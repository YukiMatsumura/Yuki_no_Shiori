
```
! warning ! 
この投稿はAndroid 7.1 Developer Preview1 - App Shortcutの内容をもとにしています. 
今後のバージョンアップで内容が変更されている可能性にご注意ください.
```

本稿は下記ドキュメントの翻訳です. 
[https://developer.android.com/reference/android/content/pm/ShortcutManager.html](https://developer.android.com/reference/android/content/pm/ShortcutManager.html)

### ShortcutManager

> The ShortcutManager manages "launcher shortcuts" (or simply "shortcuts"). Shortcuts provide users with quick access to activities other than an application's main activity in the currently-active launcher. For example, an email application may publish the "compose new email" action, which will directly open the compose activity. The [ShortcutInfo](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html) class contains information about each of the shortcuts themselves.

`ShortcutManager`は"Launcher shortcut"（または単に"ショートカット"）を管理します.  ショートカットはランチャーにあるアプリのMainアクティビティ以外のアクティビティに素早くアクセスする手段をユーザに提供します. 例えば, E-Mailアプリケーションであれば"メールを作成する"アクションを提供し, 直接メールを作成するアクティビティを起動します. `ShortcutInfo`クラスはショートカットに関する情報を保持します. 


### Dynamic Shortcuts and Manifest Shortcuts

> There are two ways to publish shortcuts: manifest shortcuts and dynamic shortcuts.
> 
> - Manifest shortcuts are declared in a resource XML, which is referenced in the publisher application's AndroidManifest.xml file. Manifest shortcuts are published when an application is installed, and the details of these shortcuts change when an application is upgraded with an updated XML file. Manifest shortcuts are immutable, and their definitions, such as icons and labels, cannot be changed dynamically without upgrading the publisher application.
> - Dynamic shortcuts are published at runtime using the [ShortcutManager](https://developer.android.com/reference/android/content/pm/ShortcutManager.html) APIs. Applications can publish, update, and remove dynamic shortcuts at runtime.

ショートカットを提供する方法にはManifest shortcutとDynamic shortcutの2つがあります.  

	 - Manifest shortcut: XMLリソースファイルで定義されるショートカット. このリソースファイルはショートカットを提供するアプリの`AndroidManifest.xml`から参照されます. Manifest shortcutはアプリがインストールされた時やアプリがアップデートされた際に提供される. Manifest shortcutは不変で, 事前に専用のアイコンやラベルが定義され, アプリをアップデートした場合を除いてこれを動的に変更することができません.  
		 - Dynamic shortcut: `ShortcutManager`のAPIを使って動的に提供されます. アプリケーションはショートカットを動的に作成, 更新, 削除することができます. 

> Only "main" activities—activities that handle the MAIN action and the LAUNCHER category—can have shortcuts. If an application has multiple main activities, these activities will have different sets of shortcuts.
> 
> Dynamic shortcuts and manifest shortcuts are shown in the currently active launcher when the user long-presses on an application launcher icon. The actual gesture may be different depending on the launcher application.
> 
> Each launcher icon can have at most [getMaxShortcutCountPerActivity()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getMaxShortcutCountPerActivity()) number of dynamic and manifest shortcuts combined.

`Intent`の`Action.MAIN`と`Category.LAUNCHER`を受け取る"Main"なアクティビティだけがショートカットを持つことができます. アプリが複数の"Main"アクティビティを持っていた場合, ことなるショートカットのセットを持つことになります.  

Dynamic shortcutとManifest shortcutはランチャーアイコンをロングタップした際に表示されます. ショートカットが表示されるためのジェスチャーは固定ではなくランチャーアプリの実装によって異なる場合があります.  

それぞれのショートカットのランチャーアイコンはDynamic shortcutとManifest shortcutを合わせた`getMaxShortcutCountPerActivity()`の数だけ指定できる.  


### Pinning Shortcuts

> Launcher applications allow users to "pin" shortcuts so they're easier to access. Both manifest and dynamic shortcuts can be pinned. Pinned shortcuts **cannot** be removed by publisher applications; they're removed only when the user removes them, when the publisher application is uninstalled, or when the user performs the "clear data" action on the publisher application from the device's Settings application.
However, the publisher application can disable pinned shortcuts so they cannot be started. See the following sections for details.

ランチャーアプリはショートカットの"ピン留め"を容易にできる機能を提供します. Manifest shortcutとDynamic shortcut両方でピン留めは可能です. ピン留めされたショートカットはそれを提供しているアプリから削除することはできません. ピン留めされたショートカットが削除されるのはユーザによる削除操作がなされた場合や, それを提供しているアプリがアンインストールされた場合, または, アプリケーション設定から当該アプリのデータ消去をした場合です. 
一方で, ショートカットを提供しているアプリはピン留めされたショートカットを無効化してショートカットからアプリを起動させなくすることができます. 以降のセクションで詳細を説明します.  


### Updating and Disabling Shortcuts

> When a dynamic shortcut is pinned, even when the publisher removes it as a dynamic shortcut, the pinned shortcut will still be visible and launchable. This allows an application to have more than [getMaxShortcutCountPerActivity()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getMaxShortcutCountPerActivity()) number of shortcuts.

Dynamic shortcutがピン留めされた場合, それに相当するDynamic shortcutが削除されたとしてもピン留めされたショートカットは残り続け, 依然として起動可能です. これはアプリが`getMaxShortcutCountPerActivity()`の数以上のショートカットを持てることを意味します.  

> For example, suppose getMaxShortcutCountPerActivity() is 5:
> 
> - A chat application publishes 5 dynamic shortcuts for the 5 most recent conversations, "c1" - "c5".
> - The user pins all 5 of the shortcuts.
> - Later, the user has started 3 additional conversations ("c6", "c7", and "c8"), so the publisher application re-publishes its dynamic shortcuts. The new dynamic shortcut list is: "c4", "c5", "c6", "c7", and "c8". The publisher application has to remove "c1", "c2", and "c3" because it can't have more than 5 dynamic shortcuts.
> - However, even though "c1", "c2" and "c3" are no longer dynamic shortcuts, the pinned shortcuts for these conversations are still available and launchable.
> - At this point, the user can access a total of 8 shortcuts that link to activities in the publisher application, including the 3 pinned shortcuts, even though it's allowed to have at most 5 dynamic shortcuts.
> - The application can use [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) to update any of the existing 8 shortcuts, when, for example, the chat peers' icons have changed.

例えば, getMaxShortcutCountPerActivity()が5と仮定すると...

 - チャットアプリは5つのDynamic shortcutを提供し, 間近の5つのチャット"c1" - "c5"を提供する
 - ユーザはそれらのDynamic shortcutを5つ全てピン留めする
 - そのあと, ユーザは新たに3つのチャットを開始する（"c6", "c7"と"c8"）. チャットアプリはこれら3つのチャットをDynamic shortcutとして再登録する. 新しいDynamic shortcutのリストは"c4", "c5", "c6", "c7", "c8"になる. この時, ショートカットを提供するチャットアプリは5つ以上のショートカットを持つことができないので"c1", "c2", "c3"のショートカットを削除する必要がある.
 - しかし, "c1", "c2", "c3"のチャットルームが使われなくなり, Dynamic shortcutから削除されたとしても, ピン留めされたそれらのチャットは依然として有効で, そこからアプリを起動しようとすることもできる. 
 - ここでのポイントは, ユーザは合計で8つのショートカットにアクセスでき, そのうちの3つはピン留めされたショートカットであり, 5つはDynamic shortcutであるということ. 
 - アプリは`updateShortcuts(List)`を使って登録済みの8つのショートカットを更新することができる（例えばチャットルームのアイコン変更など）

> The [addDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) and [setDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) methods can also be used to update existing shortcuts with the same IDs, but they cannot be used for updating non-dynamic, pinned shortcuts because these two methods try to convert the given lists of shortcuts to dynamic shortcuts.

`addDynamicShortcuts(List)`と`setDynamicShortcuts(List)`メソッドは既に登録済みの同じIDのショートカットを更新することができます.  しかし, Dynamic shortcutでないものや, ピン留めされたショートカットは更新することができません. これらのメソッドは引数で与えられたショートカットリストをDyamic shortcutに変換することを試みるものであるからです.  

#### Disabling Manifest Shortcuts

> When an application is upgraded and the new version no longer uses a manifest shortcut that appeared in the previous version, this deprecated shortcut will no longer be published as a manifest shortcut.
> If the deprecated shortcut is pinned, then the pinned shortcut will remain on the launcher, but it will be disabled automatically. Note that, in this case, the pinned shortcut is no longer a manifest shortcut, but it's still **immutable** and cannot be updated using the [ShortcutManager](https://developer.android.com/reference/android/content/pm/ShortcutManager.html) APIs. 

アプリがアップデートされて, 過去バージョンでは使えていたManifest shortcutが新しいバージョンで使えなくなった場合, この使えなくなったショートカットは提供されなくなります. もしこのショートカットがピン留めされていた場合はランチャーに残り続けます. しかし使えなくなったショートカットは自動で無効化されます. 注意すべき点として, このケースにおいてピン留めされたショートカットはManifest shortcutであり, 使えなくなったもののそれは不変であり, `ShortcutManager` APIを使っても更新することができないということです.  

#### Disabling Dynamic Shortcuts

> Sometimes pinned shortcuts become obsolete and may not be usable. For example, a pinned shortcut to a group chat will be unusable when the associated group chat is deleted. In cases like this, applications should use [disableShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#disableShortcuts(java.util.List%3Cjava.lang.String%3E)), which will remove the specified dynamic shortcuts and also make any specified pinned shortcuts un-launchable. The [disableShortcuts(List, CharSequence)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#disableShortcuts(java.util.List%3Cjava.lang.String%3E, java.lang.CharSequence)) method can also be used to disabled shortcuts and show users a custom error message when they attempt to launch the disabled shortcuts.

ある時, ピン留めされたDynamic shortcutが使われなくなり無効化された場合. 例えば所属するチャットグループが削除されて使えなくなったようなケースで	は`disableShortcuts(List)`を使用すべきです. これによって特定のショートカットを削除して, ピン留めされたショートカットであっても起動できなくする（無効化する）ことができます. `disableShortcuts(List, CharSequence)`メソッドはショートカットを無効化して, これを起動しようとした時に好きなエラーメッセージを表示させることができます. 


### Publishing Manifest Shortcuts

> In order to add manifest shortcuts to your application, first add `<meta-data android:name="android.app.shortcuts" />` to your main activity in AndroidManifest.xml:

アプリにManifest shortcutを追加するには, まずAndroidManifest.xmlで`<meta-data android:name="android.app.shortcuts" />`をMainアクティビティに追加します. 

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
   package="com.example.myapplication">
   <application . . .>
     <activity android:name="Main">
       <intent-filter>
         <action android:name="android.intent.action.MAIN" />
         <category android:name="android.intent.category.LAUNCHER" />
       </intent-filter>
       <meta-data android:name="android.app.shortcuts" android:resource="@xml/shortcuts"/>
     </activity>
   </application>
 </manifest>
```

> Then, define your application's manifest shortcuts in the res/xml/shortcuts.xml file:

そして, アプリのManifest shortcutを`/res/xml/shortcuts.xml`ファイルとして定義します.  

```xml
 <shortcuts xmlns:android="http://schemas.android.com/apk/res/android" >
   <shortcut
     android:shortcutId="compose"
     android:enabled="true"
     android:icon="@drawable/compose_icon"
     android:shortcutShortLabel="@string/compose_shortcut_short_label1"
     android:shortcutLongLabel="@string/compose_shortcut_long_label1"
     android:shortcutDisabledMessage="@string/compose_disabled_message1"
     >
     <intent
       android:action="android.intent.action.VIEW"
       android:targetPackage="com.example.myapplication"
       android:targetClass="com.example.myapplication.ComposeActivity" />
     <!-- more intents can go here; see below -->
     <categories android:name="android.shortcut.conversation" />
   </shortcut>
   <!-- more shortcuts can go here -->
 </shortcuts>
```

> The following list includes descriptions for the different attributes within a manifest shortcut:
> 
> _android:shortcutId_
> Mandatory shortcut ID
> 
> _android:enabled_
> Default is true. Can be set to false in order to disable a manifest shortcut that was published in a previous version and and set a custom disabled message. If a custom disabled message is not needed, then a manifest shortcut can be simply removed from the XML file rather than keeping it with enabled="false".
> 
> _android:icon_
> Shortcut icon.
> 
> _android:shortcutShortLabel_
> Mandatory shortcut short label. See [setShortLabel(CharSequence)](https://developer.android.com/reference/android/content/pm/ShortcutInfo.Builder.html#setShortLabel(java.lang.CharSequence)).
> 
> _android:shortcutLongLabel_
> Shortcut long label. See [setLongLabel(CharSequence)](https://developer.android.com/reference/android/content/pm/ShortcutInfo.Builder.html#setLongLabel(java.lang.CharSequence)).
> 
> _android:shortcutDisabledMessage_
> When android:enabled is set to false, this attribute is used to display a custom disabled message.
> 
> _intent_
> Intent to launch when the user selects the shortcut. android:action is mandatory. See [Using intents](https://developer.android.com/guide/topics/ui/settings.html#Intents) for the other supported tags. You can provide multiple intents for a single shortcut so that an activity is launched with other activities in the back stack. See [TaskStackBuilder](https://developer.android.com/reference/android/app/TaskStackBuilder.html) for details.
> 
> _categories_
> Specify shortcut categories. Currently only [SHORTCUT_CATEGORY_CONVERSATION](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#SHORTCUT_CATEGORY_CONVERSATION) is defined in the framework.

Manifest shortcutで定義されるそれぞれの属性について:

_android:shortcutId_
ショートカットID（必須の属性）

_android:enabled_
デフォルト値は`true`. これを`false`に設定することで過去バージョンで提供していたManifest shortcutが今バージョンから提供されなくなった場合の無効化を行い, カスタムエラーメッセージを設定することができる. もし, カスタムエラーメッセージが必要ない場合は単純にこのショートカット定義をXMLから削除すればよい. 引き続きショートカットを保持しておきたいなら`enabled="false"`とする.  

_android:icon_
ショートカットアイコン

_android:shortcutShortLabel_
短いショートカットラベル（必須の属性）. setShortLabel(CharSequence)を参照.  

_android:shortcutLongLabel_
長いショートカットラベル. setLongLabel(CharSequence)を参照.  

_android:shortcutDisabledMessage_
`android:enabled`に`false`が設定されている場合, この属性はカスタムエラーメッセージとして使用される.  

_intent_
ユーザがショートカットを選択した際に起動されるIntent. `android:action`は必須の属性である. [Using intents](https://developer.android.com/guide/topics/ui/settings.html#Intents)に`intent`タグがサポートする他の属性が記載されている. あなたは1つのショートカットに対してmultiple-intentを設定することができる. これによりバックスタックに積むアクティビティを指定しつつアクティビティを起動することができる. `TaskStackBuilder`により詳細が記載されている.  

_categories_
ショートカットのカテゴリを指定する. 現状では`SHORTCUT_CATEGORY_CONVERSATION`のみが定義されている.  


### Publishing Dynamic Shortcuts

> Applications can publish dynamic shortcuts with [setDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) or [addDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)). The [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) method can also be used to update existing, mutable shortcuts. Use [removeDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#removeDynamicShortcuts(java.util.List%3Cjava.lang.String%3E)) or [removeAllDynamicShortcuts()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#removeAllDynamicShortcuts()) to remove dynamic shortcuts.  
> 
> Example:

アプリはDynamic shortcutを`setDynamicShortcuts(List)`か`addDynamicShortcuts(List)`で登録することができます. `updateShortcuts(List)`メソッドは既に登録済みの変更可能なショートカットの更新に使うことができます.  `removeDynamicShortcuts(List)`か`removeAllDynamicShortcuts()`はDynamic shortcutの削除に使えます.  
下記はこれらの使用例です:  

```java
ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);

 ShortcutInfo shortcut = new ShortcutInfo.Builder(this, "id1")
     .setIntent(new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.mysite.com/")))
     .setShortLabel("Web site")
     .setLongLabel("Open the web site")
     .setIcon(Icon.createWithResource(context, R.drawable.icon_website))
     .build();

 shortcutManager.setDynamicShortcuts(Arrays.asList(shortcut));
```


### Shortcut Intents

> Dynamic shortcuts can be published with any set of [Intent](https://developer.android.com/reference/android/content/Intent.html#addFlags(int)) flags. Typically, [FLAG_ACTIVITY_CLEAR_TASK](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TASK) is specified, possibly along with other flags; otherwise, if the application is already running, the application is simply brought to the foreground, and the target activity may not appear.
> The [setIntents(Intent[])](https://developer.android.com/reference/android/content/pm/ShortcutInfo.Builder.html#setIntents(android.content.Intent[])) method can be used instead of [setIntent(Intent)](https://developer.android.com/reference/android/content/pm/ShortcutInfo.Builder.html#setIntent(android.content.Intent)) with [TaskStackBuilder](https://developer.android.com/reference/android/app/TaskStackBuilder.html) in order to launch an activity with other activities in the back stack. When the user selects a shortcut to load an activity with a back stack, then presses the back key, a "parent" activity will be shown instead of the user being navigated back to the launcher.

Dynamic shortcutは`Intent`フラグを指定して登録することができます. 一般的には`FLAG_ACTIVITY_CLEAR_TASK`が他のフラグと一緒に指定されます. もしアプリケーションが既に実行中であれば単に前面に遷移して, 起動ターゲットであったアクティビティは起動されてきません.  

`setIntents(Intent[])`メソッドはバックスタックにあらかじめアクティビティ順序を定義して起動する`TaskStackBuilder`と`setIntent(Intent)`の組み合わせの代わりに使うことができます. ユーザがショートカットを選択した時に, アクティビティをバックスタック付きで起動することができ, バックキーでアクティビティを破棄した場合に, ランチャーではなく親アクティビティを表示させるようなことができます. 

> Manifest shortcuts can also have multiple intents to achieve the same effect. In order to associate multiple [Intent](https://developer.android.com/reference/android/content/Intent.html) objects with a shortcut, simply list multiple <intent> elements within a single <shortcut> element. The last intent specifies what the user will see when they launch a shortcut.
> 
> Manifest shortcuts cannot have custom intent flags. The first intent of a manifest shortcut will always have [FLAG_ACTIVITY_NEW_TASK](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK) and [FLAG_ACTIVITY_CLEAR_TASK](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TASK) set. This means, when the application is already running, all the existing activities will be destroyed when a manifest shortcut is launched. If this behavior is not desirable, you can use a trampoline activity, or an invisible activity that starts another activity in [onCreate(Bundle)](https://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)), then calls [finish()](https://developer.android.com/reference/android/app/Activity.html#finish()). The first activity should include an attribute setting of android:taskAffinity="" in the application's AndroidManifest.xml file, and the intent within the manifest shortcut should point at this first activity.

Manifest shortcutも同じ効果をもたらすmultiple-intentを持つことができます. multiple-intentをショートカットに指定するには, 複数の`<intent>`要素を1つの`<shortcut>`要素に定義します. 最後に定義されたIntentがショートカット選択時にユーザから見えるアクティビティとなります.  

Manifest shortcutはintentフラグを好きに指定することができません. まず, Manifest shortcutのintentは常に`FLAG_ACTIVITY_NEW_TASK`と`FLAG_ACTIVITY_CLEAR_TASK`のフラグがセットされます. これは, Manifest shortcutから起動される場合, アプリが実行中であれば既にあるアクティビティが破棄されてからショートカットによるアクティビティが起動されることを意味しています. この動作を望まないのであれば, トランポリンアクティビティ（訳注：起動Intentをハンドルする目的だけのアクティビティ）を使うか, 透明アクティビティから別のアクティビティを`onCreate(Bundle)`で起動して即`finish()`するかです. 最初のアクティビティの属性には`android:taskAffinity=""`を含めておく必要があります. そして, ショートカットからIntentでこのアクティビティを指定して起動します.  


### Showing New Information in a Shortcut

> In order to avoid confusion, you should not use [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List<android.content.pm.ShortcutInfo>)) to update a shortcut so that it contains conceptually different information.
> 
> For example, a phone application may publish the most frequently called contact as a dynamic shortcut. Over time, this contact may change; when it does, the application should represent the changed contact with a new shortcut that contains a different ID, using either [setDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) or [addDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)), rather than updating the existing shortcut with [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)). This is because when the shortcut is pinned, changing it to reference a different contact will likely confuse the user.
> 
> On the other hand, when the contact's information has changed, such as the name or picture, the application should use [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) so that the pinned shortcut is updated too.

混乱を避けるために`updateShortcuts(List)`を使って異なる情報の内容でショートカットを更新するべきではありません. 例えば, Phoneアプリはよく連絡する相手へのDynamic shortcutをショートカットに登録します. 時間が過ぎて, 連絡先が変更された場合, アプリは新しいショートカットと新しいショートカットIDをもって変更後の連絡先へのショートカットを登録します. この時, `updateShortcuts(List)`を使うのではなく`setDynamicShortcuts(List)`か`addDynamicShortcuts(List)`のどちらかを使うようにします.  
こうする理由は, ショートカットがピン留めされていた場合, これを変更すると異なる連絡先を参照することになるためユーザが混乱してしまいます.  

別のケースとして, 顔写真や名前といった類の連絡先情報が変更された場合には, `updateShortcuts(List)`を使ってピン留めされたショートカットを更新するべきです.  


### Shortcut Display Order

> When the launcher displays the shortcuts that are associated with a particular launcher icon, the shortcuts should appear in the following order:
> 
> - First show manifest shortcuts (if [isDeclaredInManifest()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#isDeclaredInManifest()) is true), and then show dynamic shortcuts (if [isDynamic()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#isDynamic()) is true).
> - Within each category of shortcuts (manifest and dynamic), sort the shortcuts in order of increasing rank according to [getRank()](https://developer.android.com/reference/android/content/pm/ShortcutInfo.html#getRank()).

ランチャーがショートカットを表示する際にはそれに紐づく指定のランチャーアイコンが表示されます. ショートカットの並び順は次の通りです:

 - まず最初にManifest shortcut（`isDeclaredInManifest()`が`true`）を表示し, 続いてDynamic shortcut（`isDynamic()`が`true`）を表示する.
 - それぞれのショートカットカテゴリ（Manifest shortcutとDynamic shortcut）の中で, `getRank()`の値にならって並び替えられる.

> Shortcut ranks are non-negative sequential integers that determine the order in which shortcuts appear, assuming that the shortcuts are all in the same category. Ranks of existing shortcuts can be updated with [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)); you can use [addDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) and [setDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)), too.
> 
> Ranks are auto-adjusted so that they're unique for each target activity in each category (dynamic or manifest). For example, if there are 3 dynamic shortcuts with ranks 0, 1 and 2, adding another dynamic shortcut with a rank of 1 represents a request to place this shortcut at the second position. In response, the third and fourth shortcuts move closer to the bottom of the shortcut list, with their ranks changing to 2 and 3, respectively.

ショートカットランクは負の数ではないシーケンシャルな数値で, 同じカテゴリ内でのショートカットの並び順を決定します. 登録済みのショートカットのランクは`updateShortcuts(List)`で更新することができます. `addDynamicShortcuts(List)`と`setDynamicShortcuts(List)`でも同じことが可能です.  

ショートカットランクは, ショートカットを登録したそれぞれのアクティビティの各カテゴリ（Manifest shortcutかDynamic shortcut）毎に一意な数値が自動で割り当てられます. 例えば, 3つのDynamic shortcutが0, 1, 2のランクを持っていたとして, ここに新しいランク1のDynamic shortcutを追加した場合, 新しいショートカットは2番目に配置されることになります. 3番目と4番目のショートカットはショートカットリストの下方向に移動されます. そしてそれらのランクは2と3に変更されます. 


### Rate Limiting

> Calls to [setDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)), [addDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)), and [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) may be rate-limited when called by background applications, or applications with no foreground activity or service. When you attempt to call these methods from a background application after exceeding the rate limit, these APIs return false.
Applications with a foreground activity or service are not rate-limited.
> 
> Rate-limiting will be reset upon certain events, so that even background applications can call these APIs again until the rate limit is reached again. These events include the following:
> 
> - When an application comes to the foreground.
> - When the system locale changes.
> - When the user performs an "inline reply" action on a notification.
> 
> When rate-limiting is active, [isRateLimitingActive()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#isRateLimitingActive()) returns true.

`setDynamicShortcuts(List)`, `addDynamicShortcuts(List)`,  `updateShortcuts(List)`のAPIをアプリがバックグランドの状態やアクティビティがないアプリ, またはサービスから呼び出す場合, 呼び出し頻度の限度に注意する必要があります. 呼び出し頻度の限度に到達した場合, これらのメソッドは`false`を返します.  

呼び出し頻度の限度は下記のイベントによりリセットされ, アプリはバックグランドなどの状態でも再びAPIを呼び出すことができるようになります. 

 - アプリがフォアグラウンド状態に遷移した
 - システムのロケールが変更された
 - ユーザがNotificationのインライン返信を行った

APIの呼び出し頻度が限度に到達している場合, `isRateLimitingActive()`は`true`を返します.  

#### Resetting rate-limiting for testing

> If your application is rate-limited during development or testing, you can use the "Reset ShortcutManager rate-limiting" development option or the following adb command to reset it:

開発やテスト中に呼び出し限度回数に到達した場合に, 設定アプリの開発者オプションから"Reset ShortcutManager rate-limiting"を選択するか, 下記の`adb`コマンドでこれを解除することができます.  

```bash
 adb shell cmd shortcut reset-throttling [ --user USER-ID ]
```

### Handling System Locale Changes

> Applications should update dynamic and pinned shortcuts when the system locale changes using the [ACTION_LOCALE_CHANGED](https://developer.android.com/reference/android/content/Intent.html#ACTION_LOCALE_CHANGED) broadcast.
> When the system locale changes, rate-limiting is reset, so even background applications can set dynamic shortcuts, add dynamic shortcuts, and update shortcuts until the rate limit is reached again.

アプリは`ACTION_LOCALE_CHANGED`のブロードキャストイベントを検知して, Dynamic shortcutとピン留めされたショートカットを更新する必要があります. ロケールが変更されるとAPIの呼び出し制限は解除されます. バックグラウンドアプリでもDynamic shortcutを追加, 変更することができるようになります. 


### Backup and Restore

> When an application has the `android:allowBackup="true"` attribute assignment included in its `AndroidManifest.xml` file, pinned shortcuts are backed up automatically and are restored when the user sets up a new device.

アプリが`AndroidManifest.xml`で`android:allowBackup="true"`を設定している場合, ピン留めされたショートカットは自動的にシステムによってバックアップされ, 新しいデバイスのセットアップ時に自動的にリストアされます. 

#### Categories of Shortcuts that are Backed Up

> - Pinned shortcuts are backed up. Bitmap icons are not backed up by the system, but launcher applications should back them up and restore them so that the user still sees icons for pinned shortcuts on the launcher. Applications can always use [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) to re-publish icons.
> - Manifest shortcuts are not backed up, but when an application is re-installed on a new device, they are re-published from the `AndroidManifest.xml` file, anyway.
> - Dynamic shortcuts are **not** backed up.
> 
> Because dynamic shortcuts are not restored, it is recommended that applications check currently-published dynamic shortcuts using [getDynamicShortcuts()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getDynamicShortcuts()) each time they are launched, and they should re-publish dynamic shortcuts when necessary.

 - ピン留めされたショートカットはバックアップされる. しかし, Bitmapアイコンはシステムにバックアップされない. ただし, ランチャーアプリはこれらをバックアップ/リストアしてユーザが同じピン留めされたショートカットを見られるようにすべきである. ショートカットの登録元アプリは`updateShortcuts(List)`を使っていつでもアイコンを再登録することができる. 
 - Manifest shortcutはバックアップされない. ただし, アプリが新しいデバイスに再インストールされるタイミングでショートカットが`AndroidManifest.xml`をもとに再登録される.
 - Dynamic shortcutは**バックアップされない**

Dynamic shortcutがリストアされない理由は, アプリが`getDynamicShortcuts()`で現在のDynamic shortcutをアクティビティが起動される際に確認して, 必要に応じてそれらを再登録することが望ましいためです. 

```java

 public class MainActivity extends Activity {
     public void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);

         ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);

         if (shortcutManager.getDynamicShortcuts().size() == 0) {
             // Application restored; re-publish dynamic shortcuts.

             if (shortcutManager.getPinnedShortcuts().size() > 0) {
                 // Pinned shortcuts have been restored.  Use updateShortcuts() to make sure
                 // they have up-to-date information.
             }
         }
     }
 }
```

#### Backup/restore and shortcut IDs

> Because pinned shortcuts are backed up and restored on new devices, shortcut IDs should be meaningful across devices; that is, IDs should contain either stable, constant strings or server-side identifiers, rather than identifiers generated locally that might not make sense on other devices.

ピン留めされたショートカットはバックアップ/リストアされます. 古いデバイス, 新しいデバイスを通してショートカットのIDは意味を持ちます. IDは永続的で変化しないか, サーバサイドで一意であるべきで, ローカルで生成されたIDは他のデバイスにおいては意味を持たない可能性があります.  

### Report Shortcut Usage and Prediction

> Launcher applications may be capable of predicting which shortcuts will most likely be used at a given time by examining the shortcut usage history data.
In order to provide launchers with such data, publisher applications should report the shortcuts that are used with [reportShortcutUsed(String)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#reportShortcutUsed(java.lang.String)) when a shortcut is selected, **or when an action equivalent to a shortcut is taken by the user even if it wasn't started with the shortcut.**

> For example, suppose a GPS navigation application supports "navigate to work" as a shortcut. It should then report when the user selects this shortcut **and** when the user chooses to navigate to work within the application itself. This helps the launcher application learn that the user wants to navigate to work at a certain time every weekday, and it can then show this shortcut in a suggestion list at the right time.

ランチャーアプリはどのショートカットがユーザに好かれるかということをショートカットの使用履歴情報から推測する機能をもっている場合があります. ショートカットを登録するアプリはショートカットが選択された場合や, **ショートカットから起動されなくてもそれがショートカットの達成しようとするアクションであれば**`reportShortcutUsed(String)`を使ってランチャーにそのことをレポートするべきです. 

例えば, GPSナビゲーションをサポートしたアプリであれば"職場への経路案内"をショートカットに登録しているかもしれません. アプリはそのショートカットが選択された時と, **さらに**ユーザがショートカットを使わずにアプリを起動して職場への経路選択アクションをとった場合にレポートするべきです. これはランチャーアプリがユーザが平日に職場への案内を期待していることを学ぶ良い機会になります. そしてそれはユーザにとって必要な時にショートカットを提案することを可能にします.  


### Launcher API

> The [LauncherApps](https://developer.android.com/reference/android/content/pm/LauncherApps.html) class provides APIs for launcher applications to access shortcuts.

`LauncherApps`クラスはランチャーアプリ向けにショートカットへアクセスする
APIを提供しています. 


### Direct Boot and Shortcuts

> All shortcut information is stored in credential encrypted storage, so no shortcuts can be accessed when the user is locked.

すべてのショートカット情報はクレデンシャル領域に格納されます. ユーザが端末をロックしている場合ショートカットにアクセスすることができません.  
