---
title: "SQLiteDatabase - コンフリクトアルゴリズム"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["android"]
published: true
---

SQLiteDatabaseの`INSERT`/`UPDATE`文でコンフリクト対応する話. 

### はじめに

SQLiteはデータのコンフリクトを解消するコンフリクトアルゴリズムをサポートしており, `CREATE TABLE`構文で`ON CONFLICT`句として指定できる. 
`INSERT`, `UPDATE`構文では文体をより自然にするために`ON CONFLICT`ではなく`OR`句として指定される. 

Androidでもこれを使うことができる. 
`SQLiteDatabase`クラスを使って`INSERT` or `UPDATE`をする際にコンフリクトアルゴリズムを指定できるメソッドがAPI Lv.8から用意されている. 

 - [SQL As Understood By SQLite](https://www.sqlite.org/lang_conflict.html) 
 - [Android Developers - SQLiteDatabase.insertWithOnConflict](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html?hl=ja)
 - [Android Developers - SQLiteDatabase.updateWithOnConflict](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html?hl=ja)

SQLiteにおけるコンフリクト要因としては`UNIQUE`, `NOT NULL`, `CHECK`, `PRIMARY KEY`といったカラム制約がある. ただし, `FOREIGN KEY`制約についてはこれが適用されない. 

### CONFLICT\_\*

Androidでは`SQLiteDatabase.CONFLICT_*`にあたる各定数で`コンフリクトアルゴリズムを指定でき, 下記がサポートされている. 

 - [CONFLICT\_ROLLBACK](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#CONFLICT_ROLLBACK) 
 - [CONFLICT\_ABORT](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#CONFLICT_ABORT) 
 - [CONFLICT\_FAIL](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#CONFLICT_FAIL) 
 - [CONFLICT\_IGNORE](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#CONFLICT_IGNORE) 
 - [CONFLICT\_REPLACE](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#CONFLICT_REPLACE) 
 - [CONFLICT\_NONE](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#CONFLICT_NONE)


`SQLiteDatabase`は内部で下記のようにSQLを構築・発行している. 
つまり, SQLiteの`ON CONFLICT` または `OR`の仕様を理解すればよい. 

```java
private static final String[] CONFLICT_VALUES = new String[]
            {"",              // 0: CONFLICT_NONE
             " OR ROLLBACK ", // 1: CONFLICT_ROLLBACK
             " OR FAIL ",     // 2: CONFLICT_FAIL
             " OR IGNORE ",   // 3: CONFLICT_IGNORE
             " OR REPLACE "}; // 4: CONFLICT_REPLACE

public long insertWithOnConflict(String table, 
                                 String nullColumnHack,
                                 ContentValues initialValues, 
                                 int conflictAlgorithm) {
        acquireReference();
        try {
            StringBuilder sql = new StringBuilder();
            sql.append("INSERT");
            sql.append(CONFLICT_VALUES[conflictAlgorithm]);
            sql.append(" INTO ");
            sql.append(table);
            sql.append('(');
            ...
```

### 検証

今回使用するテーブルスキーマは下記

```sql
sqlite> .schema test
CREATE TABLE test ( _id INTEGER PRIMARY KEY, data STRING);
```

次のテストデータを用意. 
`_id`は, あえて2をskipしておく. 

```sql
sqlite> INSERT INTO test VALUES (1, "A");
sqlite> INSERT INTO test VALUES (3, "B");
sqlite> INSERT INTO test VALUES (4, "C");
sqlite> SELECT * FROM test;
_id         data      
----------  ----------
1           A         
3           B         
4           C         
```

このテーブルに`INSERT`や`UPDATE`文を発行してコンフリクトを発生させた. 

#### CONFLICT_ROLLBACK

`OR ROLLBACK`の指定. 
コンフリクトが発生した場合, トランザクションはロールバックされる. 

トランザクションをロールバックした後, `SQLiteException`を発生させる. 
一般的に`try-catch-finally`節でデータベーストランザクションの操作を行うが, `OR ROLLBACK`を指定した場合, 発行先のSQLがロールバックまでを実行する. 
そのため, `OR ROLLBACK`句により`SQLiteException`が発生したタイミングでは**既にトランザクションが閉じている**ので注意が必要. 
すでにトランザクションが閉じられた`db`インスタンスへの`db.endTransaction()`コールは新たな`SQLiteException`を生む. 

もし`OR ROLLBACK`がトランザクション内での実行ではなく, 単発のSQLである場合は`CONFLICT_ABORT`と同じ挙動になる. 

```java
try {
  db.beginTransaction();
  // ...
  // cv1はロールバックされ, cv2は失敗, cv3はSQLiteExceptionにより実行されない.
  db.insertWithOnConflict("test", null, cv1, CONFLICT_ROLLBACK);
  db.insertWithOnConflict("test", null, cv2, CONFLICT_ROLLBACK); // CONFLICT!
  db.insertWithOnConflict("test", null, cv3, CONFLICT_ROLLBACK);
  // ...
  db.setTransactionSuccessful();
  db.endTransaction();
} catch (SQLiteException e) {
  // Conflict occur
```


#### CONFLICT_ABORT

`OR ABORT`の指定. 
コンフリクトが発生した場合, そのSQLコマンド自体が行った変更が取り消される.

SQLコマンドによる変更を取り消した後, `SQLiteException`を発生させるがロールバックは実行されない. 
これはSQLコマンドが複数行に作用する場合に効果がある. 

`OR ABORT`はコマンドに閉じて作用するため, トランザクションで全体をロールバックするようなケースでは役に立たない. 

```java
try {
  // _idを1つずつインクリメントするが, 2レコード目でコンフリクトが発生する. 
  // クエリの結果, どのレコードも変更されないで終わる. 
  db.execSQL("UPDATE OR ABORT test SET _id=_id+1");
} catch (SQLiteException e) {
  // Conflict occur
```

#### CONFLICT_FAIL

`OR FAIL`の指定. 
コンフリクトが発生した場合でも, そのSQLコマンド自体がそこまでに行った変更は取り消さない. 

コンフリクトが発生した後, `SQLiteException`を発生させるがロールバックは実行されない. 
これはSQLコマンドが複数行に作用する場合に効果がある. 
`OR ABORT`がそのコマンドの失敗を取り消すのに対して, `OR ABORT`は取り消さない. 

`OR FAIL`はコマンドに閉じて作用するため, トランザクションで全体をロールバックするようなケースでは役に立たない. 

```java
try {
  // _idを1つずつインクリメントするが, 2レコード目でコンフリクトが発生する. 
  // クエリの結果, コンフリクトが発生しなかった最初のレコードだけ変更される. 
  db.execSQL("UPDATE OR ABORT test SET _id=_id+1");
} catch (SQLiteException e) {
  // Conflict occur
```

#### CONFLICT_IGNORE

`OR IGNORE`の指定. 
コンフリクトが発生した場合でも, それを無視して処理を継続する.  
複数行の更新を行うクエリの場合, コンフリクトが発生した1行に対しては無視され, その前後の処理は取り消されない. 

コンフリクトが発生しても`SQLiteException`は発生せず, ロールバックも実行されない. 

```java
try {
  // cv1は成功, cv2はコンフリクトで失敗するがエラーは無視される. cv3は成功.
  db.insertWithOnConflict("test", null, cv1, CONFLICT_IGNORE);
  db.insertWithOnConflict("test", null, cv2, CONFLICT_IGNORE);  // CONFLICT!
  db.insertWithOnConflict("test", null, cv3, CONFLICT_IGNORE);
} catch (SQLiteException e) {
```


#### CONFLICT_REPLACE

`OR REPLACE`の指定. 
コンフリクトが発生した場合, 次のルールでコンフリクトの解消を試す. 

 1. コンフリクトの原因が`UNIQUE` or `PRIMARY KEY`制約によるものの場合, 既存の行を削除して新たに`INSERT` or `UPDATE`を試みる. 
 2. コンフリクトの原因が`NOT NULL`制約によるものの場合, `NULL`値を`default value`に置き換えて試みる
 3. コンフリクトの原因が`NOT NULL`制約によるもの, かつ, `default vlaue`が定義されていない場合, `CONFLICT_ABORT`と同じ振る舞いとする. 
 4. コンフリクトの原因が`CHECK`制約によるものの場合は`CONFLICT_ABORT`と同じ振る舞いとする. 

コンフリクトの解消が成功した場合は`SQLiteException`は発生しない. 
コンフリクトの解消が失敗した場合は`SQLiteException`が発生する. 

```java
  // cv1は成功, cv2はREPLACEされ成功, cv3は成功
  db.insertWithOnConflict("test", null, cv1, CONFLICT_REPLACE);
  db.insertWithOnConflict("test", null, cv2, CONFLICT_REPLACE);  // CONFLICT!
  db.insertWithOnConflict("test", null, cv3, CONFLICT_REPLACE);
```


#### CONFLICT_NONE

`OR`句の指定なし. 
`CONFLICT_ABORT`と同じ挙動となる. 





ちなみに`ON CONFLICT`, `OR`句はSQL標準ではない.

以上. 
