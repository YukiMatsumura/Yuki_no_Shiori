## App Shortcut - 概要

> ! warning !
> この投稿はAndroid 7.1 Developer Preview1の内容をもとにしています.
> 今後のバージョンアップで内容が変更されている可能性にご注意ください.

本稿は下記ドキュメントの翻訳です.  
https://developer.android.com/preview/shortcuts.html


## App Shortcut

> Android 7.1 allows you to define shortcuts to specific actions in your app. These shortcuts can be displayed in a supported launcher, such as the one provided with Nexus and Pixel devices. Shortcuts let your users quickly start common or recommended tasks within your app.

Android7.1でアプリが持つ特定のアクションをショートカットとして定義できるようになりました. 
これらのショートカットはこの機能をサポートしているNexusやPixelデバイスのようなランチャー上で表示されるようになります. 
ショートカットはアプリの共通的なアクションやおススメに素早くアクセスできます.  

> Each shortcut references one or more intents, each of which launches a specific action in your app when users select the shortcut. Examples of actions you can express as shortcuts include the following:

> - Navigating users to a particular location in a mapping app.
> - Sending messages to a friend in a communication app.
> - Playing the next episode of a TV show in a media app.
> - Loading the last save point in a gaming app.

それぞれのショートカットはユーザがショートカットを選択した際に特定のアクションを起動する1つ以上のintentを参照しています. 例えば次のようなランチャーショートカットを表現できます. 

 - マップアプリの特定のロケーションにナビゲートする
 - メッセージアプリで友人にメッセージを送信する
 - メディアアプリでTV番組の次のエピソードを再生する
 - ゲームアプリのセーブの最終ポイントから始める

> You can publish two different types of shortcuts for your app:

アプリは異なる2タイプのショートカットを提供することができます.  

> - _Static shortcuts_ are defined in a resource file that is packaged into an APK. Therefore, you must wait until you update your entire app to change the details of these static shortcuts.
> - _Dynamic shortcuts_ are published at runtime using the ShortcutManager API. During runtime, your app can publish, update, and remove its dynamic shortcuts.

 - _Static Shortcut_はアプリの静的なリソースファイルに定義されます. Static Shortcutを更新するにはアプリのアップデートが必要になります

![App shortcuts: Surface key actions and take users deep into your app instantly.](https://developer.android.com/preview/images/shortcuts.png)

 - _Dynamic shortcut_は`ShortcutManager`のAPIを使って実行時にショートカットを構築します. アプリ実行中にショートカットを動的に追加したり, 更新したり, 削除したりできます. 

> You can publish up to five shortcuts (static shortcuts and dynamic shortcuts combined) at a time for your app. Users, however, can copy your app's shortcuts onto the launcher, creating pinned shortcuts. Users can create and access an unlimited number of pinned shortcuts that trigger actions in your app. Your app cannot remove these pinned shortcuts, but it can disable them.

アプリはStatic shortcutとDynamic shortcutを合わせて5つまで提供することができます.  （訳注: Shortcutの上限数はShortcutManagerのAPI[getMaxShortcutCountPerActivity](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getMaxShortcutCountPerActivity())を参照）  
しかし, ユーザはアプリのショートカットをランチャーにコピーしてピン留めしておくことができます. ピン留めできる数に上限はなく, これを選択するとショートカットと同様にアプリで定義されたアクションが実行されます. アプリからそれらのピン留めされたショートカットを削除することはできませんが, 無効化することは可能です.  

> Note: Although other apps can't access the metadata within your shortcuts, the launcher itself can access this data. Therefore, these metadata should conceal sensitive user information.

Note: 他のアプリはあなたのショートカットメタデータにアクセスすることができませんが, ランチャーアプリに限ってはこれにアクセスすることが可能です. そのため, このメタデータにユーザ情報などのセンシティブなデータを含めないようにすべきです.  



### Using Static Shortcuts

> Static shortcuts should provide links to generic actions within your app, and these actions should remain consistent over the lifetime of your app's current version. Good candidates for static shortcuts include viewing sent messages, setting an alarm, and displaying a user's exercise activity for the day.

Static shortcutはアプリでの一般的なアクションを提供するべきです. また, それらのアクションは（アプリのバージョンが同じであれば）常に一貫したものであるべきです. 例えば, 送信済みメッセージの確認や, アラームの設定, その日の運動量といったようなものが推奨されます.  

> To create a static shortcut, complete the following sequence of steps:
> 
> 1. In your app's manifest file (AndroidManifest.xml), find an activity whose intent filters are set to the [android.intent.action.MAIN](https://developer.android.com/reference/android/content/Intent.html#ACTION_MAIN) action and the [android.intent.category.LAUNCHER](https://developer.android.com/reference/android/content/Intent.html#CATEGORY_LAUNCHER) category.
> 2. Add a [`<meta-data>`](https://developer.android.com/guide/topics/manifest/meta-data-element.html) element to this activity that references the resource file where the app's shortcuts are defined:
> ```xml
> <manifest xmlns:android="http://schemas.android.com/apk/res/android"
>             package="com.example.myapplication">
>   <application ... >
>     <activity android:name="Main">
>       <intent-filter>
>         <action android:name="android.intent.action.MAIN" />
>         <category android:name="android.intent.category.LAUNCHER" />
>       </intent-filter>
>       <meta-data android:name="android.app.shortcuts"
>           android:resource="@xml/shortcuts" />
>     </activity>
>   </application>
> </manifest>
> ```
> 3. Create a new resource file (res/xml/shortcuts.xml) where the app's manifest shortcuts are defined.
> 4. In this new resource file, add a <shortcuts> root element, which contains a list of <shortcut> elements. Each <shortcut> element, in turn, contains information about a static shortcut, including its icon, its description labels, and the intents that it launches within the app:
> ```xml
> <shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
>   <shortcut
>     android:shortcutId="compose"
>     android:enabled="true"
>     android:icon="@drawable/compose_icon"
>     android:shortcutShortLabel="@string/compose_shortcut_short_label1"
>     android:shortcutLongLabel="@string/compose_shortcut_long_label1"
>     android:shortcutDisabledMessage="@string/compose_disabled_message1">
>     <intent
>       android:action="android.intent.action.VIEW"
>       android:targetPackage="com.example.myapplication"
>       android:targetClass="com.example.myapplication.ComposeActivity" />
>     <!-- If your shortcut is associated with multiple intents, include them
>            here. The last intent in the list is what the user sees when they
>            launch this shortcut. -->
>     <categories android:name="android.shortcut.conversation" />
>   </shortcut>
>   <!-- Specify more shortcuts here. -->
> </shortcuts>
> ```


Static shortcutを作成するステップ:

 1. アプリの`AndroidManifest.xml`から `android.intent.action.MAIN`アクションと`android.intent.category.LAUNCHER`カテゴリを設定したインテントフィルタを持つアクティビティを探す.
 2. アクティビティに`<meta-data>`要素でショートカット情報が定義されたリソースファイルを指定します.

	```xml
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	             package="com.example.myapplication">
	  <application ... >
	    <activity android:name="Main">
	      <intent-filter>
	        <action android:name="android.intent.action.MAIN" />
	        <category android:name="android.intent.category.LAUNCHER" />
	      </intent-filter>
	      <meta-data android:name="android.app.shortcuts"
	                 android:resource="@xml/shortcuts" />
	    </activity>
	  </application>
	</manifest>
	```

 3. Static shortcut用の新しいリソースファイルを作成します（`res/xml/shortcuts.xml`）
 4. 新しいリソースファイルで, ルート要素にあたる`<shortcuts>`を定義します. この要素には子要素として`<shortcut>`要素のリストが含まれます. それぞれの`<shortcut>`要素はStatic shortcutの情報を並び順に従って定義していきます. 定義する情報はアイコン, ラベル, intent情報が含まれます.  

	```xml
	<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
	  <shortcut
	    android:shortcutId="compose"
	    android:enabled="true"
	    android:icon="@drawable/compose_icon"
	    android:shortcutShortLabel="@string/compose_shortcut_short_label1"
	    android:shortcutLongLabel="@string/compose_shortcut_long_label1"
	    android:shortcutDisabledMessage="@string/compose_disabled_message1">
	    <intent
	      android:action="android.intent.action.VIEW"
	      android:targetPackage="com.example.myapplication"
	      android:targetClass="com.example.myapplication.ComposeActivity" />
	    <!-- If your shortcut is associated with multiple intents, include them
	         here. The last intent in the list is what the user sees when they
	         launch this shortcut. -->
	    <categories android:name="android.shortcut.conversation" />
	  </shortcut>
	  <!-- Specify more shortcuts here. -->
	</shortcuts>
	```

### Using Dynamic Shortcuts

> Dynamic shortcuts should provide links to specific, context-sensitive actions within your app. These actions can change between uses of your app, and they can change even while your app is running. Good candidates for dynamic shortcuts include calling a specific person, navigating to a specific location, and viewing the current score for a specific game.

Dynamic shortcutは特定の状況に特化したリンクを提供します. このアクションはアプリの更新の必要なしに変更することが可能で, アプリ実行中であったも変更ができます. Dynamic shortcutを使用した良い例として特定の相手への音声通話や, 特定の場所をマップで開くナビゲーション, ゲームの現在のランキングやスコアを表示するといったものがあります.  

> The [ShortcutManager](https://developer.android.com/reference/android/content/pm/ShortcutManager.html) API allows you to complete the following operations on dynamic shortcuts:
> 
> - **Publish**: Use [setDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) to redefine the entire list of dynamic shortcuts, or use [addDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) to augment an existing list of dynamic shortcuts.
> - **Update**: Use the [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) method.
> - **Remove**: Remove a set of dynamic shortcuts using [removeDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#removeDynamicShortcuts(java.util.List%3Cjava.lang.String%3E)), or remove all dynamic shortcuts using [removeAllDynamicShortcuts()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#removeAllDynamicShortcuts()).

`ShortcutManager` APIでDynamic shortcutを操作することができます:

 - **Publish**: `setDynamicShortcuts(List)`でDynamic shortcutのリストを再構築します. または`addDynamicShortcuts(List)`で既存のリストに引数のショートカットリストを追加します.  
 - **Update**: `updateShortcuts(List)`を使ってDynamic shortcutを更新します
 - **Remove**: `removeDynamicShortcuts(List)`を使ってDynamic shortcutを削除するか, `removeAllDynamicShortcuts()`を使ってすべてのショートカットを削除します. 

> An example of creating a dynamic shortcut and associating it with your app appears in the following code snippet:

次のコードはアプリに連携するDynamic shortcutを作成するものです:

```java
ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);

ShortcutInfo shortcut = new ShortcutInfo.Builder(this, "id1")
    .setShortLabel("Web site")
    .setLongLabel("Open the web site")
    .setIcon(Icon.createWithResource(context, R.drawable.icon_website))
    .setIntent(new Intent(Intent.ACTION_VIEW,
                   Uri.parse("https://www.mysite.example.com/")))
    .build();

shortcutManager.setDynamicShortcuts(Arrays.asList(shortcut));
```



### Tracking Shortcut Usage

> To determine the situations during which static and dynamic shortcuts should appear, the launcher examines the activation history of shortcuts. You can keep track of when users complete specific actions within your app by calling the [reportShortcutUsed(String)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#reportShortcutUsed(java.lang.String)) method, passing in the ID of a shortcut, when either of the following events occur:
> 
> - The user selects the shortcut with the given ID.
> - The user opens the app and manually completes the action corresponding to the same shortcut.

どのStatic shortcutとDynamic shortcutを表示するかを決定するにあたり, ランチャーはそれぞれのショートカットの使用履歴を調べます. アプリはユーザが特定のアクションを終えた際に`reportShortcutUsed(String)`メソッドに関連するショートカットIDを指定することで使用履歴に残すことができます. 次のどちらかのイベントが発生した場合にこのメソッドを呼び出します. 

 - ユーザがショートカットを選択した場合
 - ユーザがショートカットを使うことなくアプリを起動し, ショートカットに関連するタスクを完了した場合


### Disabling Shortcuts

> Because users can pin any of your app's shortcuts to the device's launcher, it's possible that these pinned shortcuts could direct users to actions within your app that are out of date or that no longer exist. To manage this situation, you can disable the dynamic shortcuts that you don't want users to select by calling [disableShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#disableShortcuts(java.util.List%3Cjava.lang.String%3E)), which removes the specified dynamic shortcuts and disables any pinned copies of these dynamic shortcuts. You can also use a variation of this method, [disableShortcuts(List, CharSequence)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#disableShortcuts(java.util.List%3Cjava.lang.String%3E, java.lang.CharSequence)), to define an error message that should appear when users attempt to launch a disabled dynamic shortcut.
> 
> Note: If you remove some of your app's static shortcuts when you update your app, the system disables these shortcuts automatically.


使用期限が切れてしまったようなアクションや, すでに存在しないアクションへのショートカットがランチャーにピン留めされていることもありえます.  こういったショートカットを使用されたくない場合は`disableShortcuts(List)`でDynamic shortcutリンクを無効にすることができます. これによってDynamic shortcutは削除されて, そのコピーであるピン留めされたショートカットは無効化されます. 同じ目的で, `disableShortcuts(List, CharSequence)`を使えば無効化されたDynamic shortcutを選択した際のエラーメッセージを指定することができます. 

Note: もしアプリの	アップデートを通してStatic shortcutを削除した場合システムはそれらのショートカットを自動で無効化します.  


### Testing Shortcuts

> Several devices, including Nexus and Pixel devices, contain a default launcher that supports shortcuts. To test your app's shortcuts, install your app on a device that uses one of these supported launchers. You should then be able to perform the following actions:
> 
> - Long-tap on your app's launcher icon to view the shortcuts that you've defined for your app.
> - Tap and drag an shortcut to pin it to the device's launcher.
> 
> You can use these interactions to test the effects of adding, updating, disabling, and removing shortcuts.

NexusやPixelデバイスといったデバイスのランチャーはApp Shortcut機能をサポートしています. アプリのショートカット機能をテストするにはそういったApp Shortcut機能をサポートしたランチャーのあるデバイスで実施します. テストでは次のようなアクションをテストします:

 - アプリアイコンのロングタップでアプリが定義したショートカットを表示する
 - ショートカットをドラッグしてランチャーにピン留めする

これらのインタラクションを使ってショートカットの追加, 更新, 無効化, 削除の影響をテストします. 


### Assigning Multiple Intents

> When creating a shortcut using [ShortcutInfo.Builder](https://developer.android.com/reference/android/content/pm/ShortcutInfo.Builder.html), you can use [setIntents(Intent[])](https://developer.android.com/reference/android/content/pm/ShortcutInfo.Builder.html#setIntents(android.content.Intent[])) instead of [setIntent(Intent)](https://developer.android.com/reference/android/content/pm/ShortcutInfo.Builder.html#setIntent(android.content.Intent)). By calling the version of this method with an array parameter, you can launch multiple activities within your app when the user selects a shortcut, placing all but the last activity in the list on the [back stack](https://developer.android.com/guide/components/tasks-and-back-stack.html). If the user then decides to press the device's back button, they will see another activity in your app instead of returning to the device's launcher.

`ShortcutInfo.Builder`を使ってショートカットを作る際, `setIntent(Intent)`の代わりに`setIntents(Intent[])`を使いことができます. このメソッドでショートカットを作成すると複数のアクティビティをまとめて起動することができます. ただし, 起動されるのは最後のアクティビティのみで残りはバックスタックにリストされます. もしこの状態でユーザがバックキーを押すとランチャーの代わりにバックスタックにある順番に積まれた別のアクティビティが表示される動作になります. 
（訳注：[TaskStackBuilder](https://developer.android.com/reference/android/app/TaskStackBuilder.html)）


### Rate Limiting

> When using the [setDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)), [addDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)), and [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) methods, keep in mind that you might only be able to call these methods a specific number of times in a background app, an app with no activities or services currently in the foreground. In a production environment, you can reset this rate limiting by bringing your app to the foreground.
> 
> If you encounter rate limiting during development or testing, you can select **Reset ShortcutManager rate-limiting** within the device's Developer Options settings, or you can enter the following command in adb:
> 
> ```bash
> $ adb shell cmd shortcut reset-throttling [ --user your-user-id ]
> ```

`setDynamicShortcuts(List)`, `addDynamicShortcuts(List)`や`updateShortcuts(List)`を使う際	, アプリがバックグラウンドにいる場合や, アクティビティを持たないアプリあるいはフォアグラウンドサービスが動作している場合にこのメソッドを呼ぶ頻度に制限が設けられていることを忘れてはいけません. 

開発やテスト中にこの制限に引っかかった場合, 開発者オプションの**Reset ShortcutManager rate-limiting**を選択するか次の`adb`コマンドでこれを解除することができます.  

```bash
$ adb shell cmd shortcut reset-throttling [ --user your-user-id ]
```


### Backup and Restore

> If you allow users to back up and restore your app when changing devices by including the [android:allowBackup="true"](https://developer.android.com/guide/topics/manifest/application-element.html#allowbackup) attribute assignment in your app's manifest file, keep in mind that only pinned shortcuts are restored to the device's launcher automatically. Also, the system doesn't back up icons associated with pinned shortcuts, so you should save your shortcuts' images in your app so that it's easy to restore them on a new device.
> 
> Static shortcuts are re-published automatically, but only after the user re-installs your app on a new device. Dynamic shortcuts, on the other hand, aren't backed up, so you must include logic in your app to re-publish them in the event that a user opens your app on a new device.


`AndroidManifest.xml`で`android:allowBackup="true"`としてアプリのバックアップとリストアをユーザに許可している場合, ピン留めされたショートカットはランチャー上に自動でリストア（ピン留め）されることを覚えておいてください. また, シスてはピン留めされたショートカットに紐づくアプリアイコンをバックアップしませんので, ショートカットの画像はアプリで保持する必要があります. これで新しい端末へ簡単にリストアされることになります. 

Static shortcutはあなたのアプリが再インストールされた後で自動的に再登録されます. Dynamic shortcutは自動でバックアップされませんのでアプリが新しい端末上で起動された場合にDynamic shortcutを再登録するコードを含めるようにしてください.  


### Best Practices

> When designing and creating your app's shortcuts, you should follow these guidelines:

> **Follow the shortcuts design guidelines**
> To make your app's shortcuts visually consistent with the shortcuts used for system apps, follow the [Shortcuts Design Guidelines](https://material.google.com/style/icons.html#icons-launcher-shortcut-icons).

**ショートカットのデザインガイドラインを参考にする**  
アプリのショートカットをシステムアプリが使っているような見た目に統一するためにも, ショートカットのデザインガイドラインを参考にしてください.  

> **Publish only four distinct shortcuts**
> Although the API currently supports a combination of up to five static shortcuts and dynamic shortcuts for your app at any given time, it's recommended that you publish only four distinct shortcuts at any time to improve the shortcuts' visual appearance in the launcher.

**4つの異なるショートカットを登録する**  
APIは現状Static shortcutとDynamic shortcutあわせて5まで登録できるようサポートしてはいますが, ランチャーでの見た目を良くするためにもショートカットは異なる4つまでに留めることを推奨します. 

> **Limit shortcut description length**
> Space is limited within the menu that shows your app's shortcuts in the launcher. When possible, limit the length of the "short description" of a shortcut to 10 characters, and limit the length of the "long description" to 25 characters.

**ショートカットの説明文字列長**  
ランチャーで表示できるショートカットのメニュー表示領域には限りがあります. ショートカットの短い説明文には10文字, 長い説明分でも25文字以内に収める必要があります. 
（訳注：全角ではさらに文字数を減らす必要がある）

> **Maintain shortcut and action usage history**
> For each shortcut that you create, consider the different ways in which a user can accomplish the same task directly within your app. Remember to call [reportShortcutUsed(String)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#reportShortcutUsed(java.lang.String)) in each of these situations so that the launcher maintains an accurate history of the actions representing your shortcuts.

**ショートカットと使用履歴の管理**
あなたが作成したすべてのショートカットについて, ショートカットを使わずにユーザがアプリを直接起動する等の異なる方法でショートカットが成そうとするものと同じ内容のタスクを達成した場合, [reportShortcutUsed(String)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#reportShortcutUsed(java.lang.String))を呼ぶことを忘れないでください. ランチャーはショートカットを提供するために正確なアクションの使用履歴を管理しています.  


> **Update shortcuts only when their meaning is retained**
> When changing dynamic shortcuts, call [updateShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#updateShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) only when changing the information of a shortcut that has retained its meaning. Otherwise, you should use [addDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#addDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) or [setDynamicShortcuts(List)](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#setDynamicShortcuts(java.util.List%3Candroid.content.pm.ShortcutInfo%3E)) instead to create a shortcut with a new ID that represents a new meaning.
> 
> For example, if you created a shortcut for navigating to a supermarket, it would be appropriate to just update the shortcut if the name of the supermarket changed but its location stayed the same. If the user started shopping at a different supermarket location, however, it would be better to create a new shortcut.

**ショートカットを更新する際にその意味を変えない**
`updateShortcuts(List)`でDynamic shortcutを変更する際, 変更するのはショートカットの情報だけにして, そのショートカットの意味は変更しないでください. 意味を変更する場合には`addDynamicShortcuts(List)`か`setDynamicShortcuts(List)`を代わりに使って新しい意味のショートカットを新しいショートカットIDで作り直します.  

> **Dynamic shortcuts aren't preserved during backup and restore**
> Dynamic shortcuts aren't preserved when a device undergoes a backup and restore operation. For this reason, it's recommended that you check the number of objects returned by [getDynamicShortcuts()](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getDynamicShortcuts()) each time you launch your app and re-publish dynamic shortcuts as needed, as shown in the following code snippet:

**Dynamic shortcutはバックアップ/リストアで引き継がれない**
Dynamic shortcutの情報は端末のバックアップ/リストア操作で引き継がれません. この理由は, アプリが起動された際に`getDynamicShortcuts()`メソッドでショートカットの数を確認して, Dynamic shortcutを必要に応じて再登録することが望ましいからです.  
次のコードはその一例です: 

```java
public class MainActivity extends Activity {
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ShortcutManager shortcutManager =
                getSystemService(ShortcutManager.class);

        if (shortcutManager.getDynamicShortcuts().size() == 0) {
            // Application restored. Need to re-publish dynamic shortcuts.
            if (shortcutManager.getPinnedShortcuts().size() > 0) {
                // Pinned shortcuts have been restored. Use
                // updateShortcuts(List) to make sure they
                // contain up-to-date information.
            }
        }
    }
    // ...
}
```

