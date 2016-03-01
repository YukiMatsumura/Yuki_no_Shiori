# Dagger2

本稿はDI FrameworkとDagger2.0の概要になります. 
対象読者は下記です. 

 - DI Frameworkを使ったことがない人.
 - Dagger2の初学者

スライドの下書きから起こしたものなのであしからず... 


### 依存性

 - 具象クラスとの関連は結合度を高める
 - インタフェースに依存させたいが, "`new`"が具象クラスへの依存性を生む 

```java
GitHubStore store = new GitHubDatabase();
```


#### 制御の反転

依存性解決の方向を反転させれば解決する. 

```text
GitHub => new GitHubDatabase

　  ↓ 反転 ↓

GitHub <= new GitHubDatabase
```

GitHubクラスが依存オブジェクトを決めるのではなく, GitHubクラスの依存オブジェクトを外から指定する. 

```java
class GitHub {
    // GitHubクラス自身が依存性を生む
    private GitHubStore store = new GitHubDatabase();


　↓ refactoring↓


class GitHub {
    GitHub(GitHubStore store) {...}
}

class Client { ...
    public doSomething() {
        // GitHubクラスの利用側が依存性を注入する
        new GitHub(new GitHubDatabase());
```

 - 制御の反転. Inversion of Control
 - ハリウッドの原則. Hollywood Principle


#### new, new, new...

制御を反転させるだけではオブジェクトを生成するコードがプロジェクト中に散在する. 

```java
ClientA:
    new GitHub(new GitHubDatabase());

ClientB:
    setDatabase(new GitHubDatabase());

ClientC:
    create(new GitHubDatabase());
```

これは下記の問題を引き起こす. 

 - 柔軟性がない
 - テストしづらい

もし, 永続化先をディスク領域に格納されるデータベースから, オンメモリキャッシュに変更したい場合, `new GitHubDatabase()`のコードを `new GitHubMemcached()`に置き換えなければならない. 

```java
ClientA:
    new GitHub(new GitHubMemcached());

ClientB:
    setDatabase(new GitHubMemcached());

ClientC:
    build(new GitHubMemcached());
```

これは簡易な例で, 実際には生成オブジェクトの初期化や組み立てといったコードが散在することになり, それらを全て置換するのには骨が折れる. 
アーキテクチャのレイヤー境界面も曖昧になり, テストの際に実装の詳細を差し替えられない. 


#### Factory Pattern

生成処理をFactoryに委譲することで, これらの問題を軽減できる. 

```java
class GitHubStoreFactory {
    public GitHubStore get() {
        return new GitHubDatabase();
    }
}

ClientA:
    new GitHub(GitHubStoreFactory.get());

ClientB:
    setDatabase(GitHubStoreFactory.get());

ClientC:
    build(GitHubStoreFactory.get());
```

```text
Factory委譲前: 
    [Client]---->[GitHubStore]
       |
       ---new--->[GitHubDatabase]


Factory委譲後: 
    [Client]---->[GitHubStore]<---[GitHubDatabase]
       |                               ↑
       ---get--->[Factory]-----new------
```

Factoryを経由すればClientはインタフェース(`GitHubStore`)にだけ依存する. 


#### Factoryの問題

Factory Patternは問題の全てを解決してくれない. 
Factory関連のクラスは次の問題に悩まされる. 

 - 数多のクラスからオブジェクト生成を委譲され肥大化する. 
 - オブジェクトの生成順序, 構築方法にも関心を持つ責務過多. 
 - 膨大なボイラープレートコードが出来上がる 
 - SharedObject? Singleton? 
 - Lifecyle, Scopeの管理が必要になることもある

下記のようなコードがプロジェクト中に散在する. 

```java
Factory factory = new Cupcake.Factory(type, key);
factory.get();
Factory factory = new Donut.Factory(type, key);
factory.get();
Factory factory = new Eclair.Factory(type, key);
factory.get();
Factory factory = new Froyo.Factory(type, key);
factory.get();
Factory factory = new Gingerbread.Factory(type, key);
factory.get();
Factory factory = new Honeycomb.Factory(type, key);
factory.get();
```


### DI Framework

こうした問題を軽減, 解決してくれるのが依存性注入に特化したDI Frameworkである. 


#### DI Frameworkのメリット

 - アーキテクチャのレイヤーをきれいに分離できる
 - 依存オブジェクトの管理を委譲できる
 - 柔軟なソフトウェアになる
 - ボイラープレートコードを排除できる
 - テストしやすいソフトウェアになる

#### DI Frameworkのデメリット

 - ラーニングコストがかかる
 - 自動生成コード含め, クラスの数が多くなる
 - Frameworkの特性にあわせた依存性の管理


### Dagger2

DI Frameworkの実装としてDagger2がある. 
Dagger2の特徴は下記の通り. 

 - DI Framework for Java & Android
 - No XML Configuration.
 - 高速
 - Annotation Processingベースでデバッグしやすい
 - コンパイル時に依存性の検証を行う
 - Googleがメンテナ


#### 依存性の要求

Dagger2に依存オブジェクトを要求するには, 依存性を注入したい箇所に`@Inject`でアノテートする. 

```java
GitHubDatabase store = new GitHubDatabase();

　↓ refactoring ↓

// フィールドstoreへの依存性注入をDagger2へ要求する
@Inject GitHubDatabase store;
```

```java
GitHub(new GitHubDatabase()) {...}

　↓ refactoring ↓

// 引数storeへの依存性注入をDagger2へ要求する
@Inject GitHub(GitHubDatabase store) {...}
```

依存オブジェクトの要求を受けたDagger2は適切なオブジェクトをそこに注入する. 

```java
@Inject GitHubDatabase store;

               ↑ inject ↑

// Dagger2はGitHubDatabaseを生成して依存性を注入する
Dagger2: new GitHubDatabase();
```

```java
@Inject GitHub(GitHubDatabase store) {...} 

               ↑ inject ↑

// Dagger2はGitHubDatabaseを生成して依存性を注入する
Dagger2: new GitHubDatabase();
```

依存性の注入には種類がある. 

 - Constructor Injection
 - Field Injection
 - (Setter Injection) Dagger2では未サポート


#### 依存性の解決

Dagger2は依存オブジェクトをどのように解決しているのか. 

```java
@Inject GitHub(GitHubDatabase store) {...} 

                  ↑ inject ↑

Dagger2: new GitHubDatabase();
         ~~~~~~~~~~~~~~~~~~~~~
         Whats happen!?
```

依存性の要求は, Dagger2管理下にある依存オブジェクトのコレクションから選択・解決される.
Constructorに`@Inject`アノテートをつけるとDagger2がこれを管理対象として収集する.

```java
class GitHubDatabase {
    // Dagger2管理対象として登録
    @Inject GitHubDatabase() {...}
}
```

```text
@Inject GitHubDatabase() {...}
  |
  | <登録>
  ↓
Dagger2
  |
  | <注入>
  +-------> @Inject GitHub(GitHubDatabase db)
  |
  | <注入>
  +-------> @Inject GitHubDatabase db;
```


#### 依存性の充足

Constructorへの`@Inject`だけでは依存性を解決できないケースがある. 

 - インタフェースへの注入(具体化)
 - プロジェクト管理外クラスの注入
 - オブジェクトの構築を伴う生成, 及び注入

これらを含む依存性を充足させるには`Provider`と呼ばれるファクトリメソッドを作る. 


#### @Provides

 - インタフェースへの注入(具体化)

```java
@Provides
GitHubStore provideGitHubStore() {
  return new GitHubDatabase();
}

OR...

@Provides
GitHubStore provideGitHubStore(GitHubDatabase store) {
  return store;
}
```

 - プロジェクト管理外クラスの注入
 - オブジェクトの構築を伴う生成, 及び注入

```java
@Provides
Retrofit provideGitHubRetrofit() {
  return new Retrofit.Builder()
      .baseUrl("https://api.github.com")
      .addConverterFactory(GsonConverterFactory.create())
      .build();
}
```


#### @Module

`@Provides`はモジュールクラス(`@Module`)のメソッドとして定義する. 

```java
@Module
class ApplicationModule {
    @Provides
    GitHubStore provideGitHubStore(GitHubDatabase store) {
        return store;
    }
}
```


#### Building the Graph

依存性のコレクションはGraphと呼ばれる. 

```text
 RepositoryViewer
     |
     |
   GitHub
     |
     |--------------------+
     |                    |
 GitHubWebApi       GitHubDatabase
     |                    |
     |                    |
  Retrofit               Orma
```

Graphの設計図としてコンポーネントクラス(`@Component`)が必要になる. 

```java
@Component(modules=ApplicationModule.class)
interface ApplicationComponent {...}
```

GraphはComponent単位で生成・管理される. 
Dagger2が管理するGraphにアクセスするにはComponentを経由する. 

```java
// Graphの取得. 
// ApplicationComponentインスタンスに依存オブジェクトが保持されている.
ApplicationComponent component 
    = DaggerApplicationComponent.builder()
        .applicationModule(new ApplicationModule())
        .build();
```


#### Sample code.

[GitHub - Dagger2Sample](https://github.com/YukiMatsumura/Dagger2Sample)

 - app : Dagger2を使った基本的なsample
 - subcomponent : Subcomponentとdependenciesのsample(後述)


### テストとアーキテクチャ

DIが促進するもの.

 - アーキテクチャにおける"レイヤー"を綺麗に分離することができる
 - レイヤーが独立し, レイヤーごと差し替えるといったことが容易 

```text
 RepositoryViewer
     |
   GitHub
     |
     |--------------------+
     |                    |
 GitHubWebApi       GitHubDatabase
     |                    |
  Retrofit               Orma


  ↓ Databaseをやめてオンメモリ管理 ↓


 RepositoryViewer
     |
   GitHub
     |
     |--------------------+
     |                    |
 GitHubWebApi      GitHubMemcached*
     |
  Retrofit
```

これらはテストの際に役立つ. 

 - テストは検証用モジュールで実施したい. 
 - Amazon Device Farm上ではテスト時間短縮のため, オンメモリDBで動作させたい. etc. 

上記の詳細はSample codeを参照. 


### 補足

これ以降はDagger2の補助機能.


#### Graphの操作

ComponentはGraph単位の操作を定義できる. 

 - Graphが属するScopeの宣言
 - 依存性注入のポイントを外部公開
 - 他Componentへの依存


#### Instant Injection

Graph生成後に, 特定オブジェクトの依存性を充足させる. 

```java
@Component(...)
interface ActivityComponent {
    void inject(RepositoryViewerActivity activity);
}

class RepositoryViewerActivity extends Activity {
    @Inject GitHub github;

    protected void onCreate(Bundle b) {
        ActivityComponent component 
            = DaggerActivityComponent.builder()
            ...
        component.inject(this);  // inject GitHub dependency.
    }
}
```


#### Scope

依存オブジェクトのライフサイクルを指定する. 

 - Application単位のSingleton性を持たせる
 - Activity単位のSingleton性を持たせる etc.

```java
@Singleton
class GitHubDatabase {...}

// Custom Scopeも定義可能
@ActivityScope
class GitHub {...}
```


#### Qualifier

依存性の注入先に識別子を付ける. 
同じ型の依存性解決に使用される. 

```java
public GitHub(...,
    @Named("executionScheduler") Scheduler executionScheduler,
    @Named("postScheduler") Scheduler postScheduler) {

@Named("executionScheduler")
@Provides @ActivityScope
public Scheduler provideExecutionScheduler() {
    return Schedulers.newThread();
}

@Named("postScheduler")
@Provides @ActivityScope
public Scheduler providePostScheduler() {
    return AndroidSchedulers.mainThread();
}
```


#### Lazy injections

 - 依存性の注入タイミングをオブジェクト取得時まで遅らせる遅延初期化

```java
@Inject Lazy<GitHub> github;

// このタイミングで依存オブジェクトが初期化される
github.get().findRepository(...);
```


#### Provider injections

 - 依存性注入の都度newするnon-cached指定

```java
@Provides LocalTime provideLocalTime() {
    return LocalTime.now();
}

@Inject Provider<LocalTime> localTimeProvider;

localTimeProvider.get();  // 常に最新の時刻が取れる.
```


#### Subcomponent

 - ComponentAとComponentBに親子関係を持たせる
 - ComponentA＋ComponentBのGraphをつくる

```java
@Component(...)
public interface ParentComponent {
    ChildComponent newChildComponent(...);
}

@Subcomponent(...)
public interface ChildComponent {...}
```


#### dependencies

 - ComponentAとComponentBに使用関係を持たせる
 - ComponentA＋ComponentBのGraphをつくる

```java
@Component(dependencies = DependeeComponent.class, ...)
public interface DependerComponent {...}

@Component(...)
public interface DependeeComponent {...}
```

以上. 