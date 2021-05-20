# NavHostFragmentをFragmentの入れ子にする時はsetPrimaryNavigationFragmentを指定する

`NavHostFragment` で `app:defaultNavHost=true`を指定すればバックキー制御をNavHostFragmentに任せることができます.

```xml
    <androidx.fragment.app.FragmentContainerView
        android:name="androidx.navigation.fragment.NavHostFragment"
        app:defaultNavHost="true"
        ...
```

`NavHostFragment`をアクティビティのレイアウトに指定した時の構造は次の通りです.

```bash
Activity
  |- NavHostFragment
```

一方で, アクティビティ直下に`NavHostFragment`を配置せず, 下記のように間にフラグメントがいる場合は注意が必要です.

```bash
Activity
  |- Fragment
     |- NavHostFragment
```

この場合, フラグメントのレイアウトで`app:defaultNavHost=true`を指定しても, バックキー制御などナビゲーション周りで意図しない動作となります.

### 解決策

`NavHostFragment`を持つフラグメントを `PrimaryNavigationFragment` に設定します.

```kotlin
class HostFragment : Fragment {
    override fun onAttach(context: Context) {
        super.onAttach(context)
        parentFragmentManager.commit {
            setPrimaryNavigationFragment(this@HostFragment)
        }
```

[FragmentTransaction.setPrimaryNavigationFragment](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction?hl=en#setPrimaryNavigationFragment(androidx.fragment.app.Fragment))

### `app:defaultNavHost`

`NavHostFragment`は`app:defaultNavHost=true`を指定されると, 自身の`onAttach`で同様に `setPrimaryNavigationFragment(this)` を設定します.

[NavHostFragment.ktの該当行 - GitHub](https://github.com/androidx/androidx/blob/6b9af68666080b59a1d58538925d19a4edb2b1ac/navigation/navigation-fragment/src/main/java/androidx/navigation/fragment/NavHostFragment.kt#L108)

### `setPrimaryNavigationFragment`

プライマリナビゲーションフラグメントに指定されると, バックナビゲーションなどをハンドリングできるようになります.
`app:defaultNavHost=true`を指定するだけで, `NavHostFragment`がバックナビゲーションをうまく制御できるのはこのためです.

プライマリナビゲーションフラグメントはフラグメントマネージャのインスタンス毎に１つしか設定できません.

[FragmentManager.setPrimaryNavigationFragment - GitHub](https://github.com/androidx/androidx/blob/6b9af68666080b59a1d58538925d19a4edb2b1ac/fragment/fragment/src/main/java/androidx/fragment/app/FragmentManager.java#L3398)

`NavHostFragment`を複数管理する場合, `app:defaultNavHost=true` な`NavHostFragment`は1つにしなければならない理由でもあります.

### NavHostFragmentをFragmentの入れ子にする

フラグメントがプライマリナビゲーションフラグメントと判定されるには, 親フラグメントがいる場合, 関連するフラグメントマネージャのプライマリナビゲーションフラグメントに指定されている必要があります.

つまり, 次の構造では`NavHostFragment`の親フラグメント/親フラグメントマネージャがいないので, `NavHostfragment`がプライマリナビゲーションフラグメントになります.

```bash
Activity
  |- NavHostFragment
```

しかし, 次の構造では`NavHostFragment`に親フラグメントがおり, その親が`setPrimaryNavigationFragment`として指定されていない場合, 子である`NavHostFragment`もプライマリナビゲーションフラグメントの条件を満たしません.

```bash
Activity
  |- Fragment
     |- NavHostFragment
```

そのため, 親フラグメントは次のようなコードで自身をプライマリナビゲーションフラグメントとして指定する必要があります.

```kotlin
class HostFragment : Fragment {
    override fun onAttach(context: Context) {
        super.onAttach(context)
        parentFragmentManager.commit {
            setPrimaryNavigationFragment(this@HostFragment)
        }
```

### 蛇足: NavHostFragmentのバックナビゲーション周りの実装

```
💡 デフォルトでフラグメントマネージャのOnBackPressedCallbackはenable=falseになっている

FragmentManager
    private final OnBackPressedCallback mOnBackPressedCallback =
            new OnBackPressedCallback(false) {
                @Override
                public void handleOnBackPressed() {
                    FragmentManager.this.handleOnBackPressed();
                }
            };

---

💡 OnBackPressedCallbackをenable=trueにするにはisPrimaryNavigationでtrueを返す必要がある

    private void updateOnBackPressedCallbackEnabled() {
        ...
        // This FragmentManager needs to have a back stack for this to be enabled
        // And the parent fragment, if it exists, needs to be the primary navigation
        // fragment.
        mOnBackPressedCallback.setEnabled(getBackStackEntryCount() > 0
                && isPrimaryNavigation(mParent));
    }
    
---

💡 PrimaryNavigationFragmentに変更があると...

Fragment
    void performPrimaryNavigationFragmentChanged() {
        boolean isPrimaryNavigationFragment = mFragmentManager.isPrimaryNavigation(this); ⭐️
        // Only send out the callback / dispatch if the state has changed
        if (mIsPrimaryNavigationFragment == null
                || mIsPrimaryNavigationFragment != isPrimaryNavigationFragment) {
            mIsPrimaryNavigationFragment = isPrimaryNavigationFragment;
            onPrimaryNavigationFragmentChanged(isPrimaryNavigationFragment); 🍣

---

💡 一方, NavHostFragmentでは...

NavHostFragment
🍣
    public void onPrimaryNavigationFragmentChanged(boolean isPrimaryNavigationFragment) {
        if (mNavController != null) {
            mNavController.enableOnBackPressed(isPrimaryNavigationFragment); 🌴

---

💡 BackPressedCallbackを有効にするにはisPrimaryNavigationFragmentがtrueである必要がある.

NavController
🌴
    void enableOnBackPressed(boolean enabled) {
        mEnableOnBackPressedCallback = enabled;  
        updateOnBackPressedCallbackEnabled();

    private void updateOnBackPressedCallbackEnabled() {
        mOnBackPressedCallback.setEnabled(mEnableOnBackPressedCallback
                && getDestinationCountOnBackStack() > 1);

---

💡 NavControllerはOnBackPressedCallbackを持っている. NavHostFragmentのバックキー制御はNavControllerの責務

    private final OnBackPressedCallback mOnBackPressedCallback =
            new OnBackPressedCallback(false) {
        @Override
        public void handleOnBackPressed() {
            popBackStack();
        }
    };
```

以上.
