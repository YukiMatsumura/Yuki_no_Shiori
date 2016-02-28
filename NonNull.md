## NonNullアノテーション

Android Studio 0.5.5から[@NonNullアノテーションがサポートされた](https://sites.google.com/a/android.com/tools/recent/androidstudio055released).  
今回はメソッドの引数に@NonNullアノテーションをつけるケースについて書いた. 

### @NonNull

@NonNullアノテーションはフィールドやメソッドの引数, メソッドのReturn値にNullを許容しないことを表明するアノテーションである. 
このアノテーションが便利な点は, @NonNullアノテーションにNullを代入したり, 戻り値のNullチェックをしたりする場合に, IDEが@NonNullの制約をCode Inspectionで示してくれるところだ. 

ただし, これはIDE付属のCode Inspection設定で報告レベルを調整可能でもあるし, @NonNullの値にNullを設定したからといってシンタックスエラーにはならない. 
@NonNullはドキュメンテーションや支援機能であって, Nullを防ぐものではない点に注意が必要である. 

下記のコードは`hoge`メソッドの引数`s`にNullを渡す恐れがある. しかし, これはCode Inspectionとして報告されない. 

```java
private String piyo;

public void foo() {
    hoge(piyo);
}

public void hoge(@NonNull String s) {
}
```

@NonNullはメソッドの呼び出し元でNullが渡らないことを保証する必要がある. 
Code Inspectionは@NonNullフィールド, メソッド引数, 戻り値の非Null性を保証するものではないからだ. 


### @NonNullの使いどころ

メソッドが引数にNullを許容せず, `NullPointerException`をスローするケースは少なくない.  
そういう時こそ@NonNullアノテーションの出番だ. メソッドの利用者に配慮する意味でも@NonNullを是非つけておきたいところだ. 

@NonNullをつければ, メソッド引数の事前条件確認としてのNullチェックはスキップしてもいいだろう. 
ただ, "もしNullが渡ってきたら"を考えておいたほうが良さそうなケースがある. 

例えば, 次のコードの場合を考えてみる. 

```java
public class Hoge {
    @NonNull private final Foo foo;
    
    public Hoge(@NonNull Foo foo) {
        this.foo = foo;
    }

    public String bar() {
        this.foo.call();
    }

    public void piyo(@NonNull Foo f) {
        f.call();
    }
// ...
```

クラス`Hoge`はコンストラクタで@NonNullな`foo`を引数にとる. 
このコンストラクタが`NullPointerException`を返すことはないが, `Hoge`クラスとしてメンバ変数`foo`がnullの状態になることを許容したくないのだ. 
それを明示するためにもコンストラクタの引数`foo`には@NonNullアノテーションを付けておいた. 

もし, `Hoge`のコンストラクタで`foo`にNullが渡った場合のことを考えると少し厄介な問題を引き起こすかもしれない. 

まず, メソッド`bar`を確認してみる. 
`this.foo`の値は同クラスのコンストラクタで@NonNullな引数`foo`で初期化されたものであり, 非Nullであることが事前条件として成立している(おまけに`final`付きだ). 
もし, 仮に`this.foo`がNullであるような事態に陥ったとしても`bar`は実行時の予期せぬ例外として`NullPointerException`をスローする. 
try-catchで例外を握りつぶすこともない. このメソッドのコードに問題はなさそうだ. 

次に, コンストラクタを確認してみる. 
このコンストラクタはシンプルで, 引数`foo`を受け取り, これでメンバー変数を初期化する. 
もし, 仮にメソッドの引数`foo`にNullが代入されてしまっても, このコンストラクタは何の異常も報告しない(にも関わらず@NonNullアノテーションを使用している). 
このケースは問題に発展するかもしれない. 

メンバ変数`foo`が意図せずNullで初期化されていることを知るのはメソッド`bar`を実行した時だ. 
クラス`Hoge`が単純な責務しかもたず, 特定のクラスとしか関連しない場合や, コンストラクタで初期化した直後に`bar`を呼ぶ場合は, `foo`にNullを渡した犯人を突き止めることができるかもしれない. 
しかし, クラス`Hoge`が複数のクラスで共有されるshared objectのような立場であった場合.
Nullで`this.foo`が初期化されるタイミングでは沈黙し, Nullを渡した犯人とは異なる運の悪い別インスタンスが`bar`を呼ぶことではじめて`NullPointerException`が報告される. 

こうなると話はややこしくなる. 
当然, `NullPointerException`のスタックトレースに犯人の痕跡は残っていない. 
@NonNullであるはずの引数`foo`に一体誰がNullを代入したのか. NonNull Code Inspectionの網をくぐり抜けた犯人を追うことになる. 

事件はすぐに解決できるかもしれないが, 事件を未然に防ぐことに努めるべきだろう.  
クラス`Hoge`のコンストラクタで何かできることはあるだろうか. 

まず@NonNullのアノテーションを外してみよう. そしてJavadocの事前条件欄にでも**"Nullは受け付けない"**旨を強調した上で, コードでもNullチェックをしてみよう. 

```java
public class Hoge {
    /**
     * @param foo can noooooooooooooot be NULL!
     */
    public Hoge(Foo foo) {
        if (foo == null) {
            throw new NullPointerException("...");
        }
        this.foo = foo;
    }
```

こうすれば望まれないNullが設定された際には即座に`NullPointerException`をスローすることができ, 犯人をスタックトレース上にあぶり出すことができるはずだ. 
だが, これだとせっかくのNonNull Code Inspectionの恩恵を受けられない. 犯人をあぶり出すことができるようになっても, 誤ってNullを設定してしまう犯人(イージーなミス)が増えてしまいそうだ. 

@NonNullアノテーションの恩恵は残しておきたい. 
なので@NonNullアノテーションはそのまま残して, Nullチェックのコードを追加してみる. 

```java
    public Hoge(@NonNull Foo foo) {
        if (foo == null) {
            throw new NullPointerException("...");
        }
        this.foo = foo;
    }
```

些か奇妙なコードにも見える. そのせいかAndroid Studioも "Condition 'foo == null' is always 'false'" と別のCode Inspectionをコメントしてきた. 
確かにその通り(そうであって欲しいの)だが, 望まない副作用を避けるためにもこの場所にはNullチェックを書いておきたい事情があるのだ. 
このまま放っておいても良いが, 常に警告されているのも気持ち良いものではないのでAndroid Studioには"そうではない"ことを伝えておく. 

```java
    public Hoge(@NonNull Foo foo) {
        // noinspection ConstantConditions
        if (foo == null) {
            throw new NullPointerException("...");
        }
        this.foo = foo;
    }
```

これで毎回 "些か奇妙なコード" をLint checkの度, 目にすることもなくなった. 

ところで, 残ったメソッド`piyo`の方は問題ないだろうか. 

```java
    public void piyo(@NonNull Foo f) {
        f.call();
    }
```

これは問題ないだろう. 引数`f`にNullを渡された場合でも潜伏期間なしに`NullPointerException`がスローされ, スタックトレースには犯人が載っているはずだ. 
`f`にNullを設定すると`NullPointerException`がスローされる正しいコードだ. 


### まとめ

@NonNullはあくまでも補助的な機能であって非Nullを保証するものではない. 
メソッドの引数にNullが渡ると`NullPointerException`が発生することを明示する場合, @NonNullアノテーションはとても役に立つ. 

本来Nullを許容しない(`NullPointerException`をスローする)ことを明示する@NonNullアノテーションだが, Nullを受け取っても異常を報告せず終了してしまう@NonNullな引数を作ることもできてしまう. 
そのようなケースでも, @NonNullが有益なケースはあるが, それによる副作用も考慮した上で, 必要なNullチェックは書いておこう. 

以上. 
