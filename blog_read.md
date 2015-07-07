## リソースの作成 ##

リソースはMVCでいうとモデルに当たる部分です。

リクエストに応じたメソッドが呼び出され結果がクライアントに返ります。リソースは独立していて、ページからだけでなくコマンドラインからも直接呼び出せます。（バッチでも利用できます）

リソース内ではリソースリクエストメソッドに応じたリソースリクエストインターフェイスを記述します。ここでいうインターフェイスはphpのinterfaceとは違います。例えばonReadメソッドを用意すればこのリソースにたいしてreadでアクセスすることができるようになります。メソッド内ではリソースに対する実リクエストを記述します。

Note:
> このブログでは`BEAR_Query`クラスというBEARのDBクエリーツールを使ってSQLを扱っていますが、PDO関数やDoctorine等他のライブラリの選択は自由です。データソースに何を(MySQL/Mongo )選択するか、その操作にどのライブラリ(`BEAR_Query`, PDO, `Zend_Db`, Doctorine ORM/DAL...）を選択するのかはユーザーやアプリケーション要求に応じて選定します。


App/Ro/Post.php
```
<?php

class App_Ro_Post extends App_Ro
{
    protected $_table = 'posts';

    public function onInject()
    {
        parent::onInject();
        $this->_query = BEAR::factory('BEAR_Query', $this->_queryConfig);
    }

     public function onRead($values)
     {
         $sql = "SELECT * FROM posts";
         $result = $this->_query->select($sql, array(), $values);
         return $result;
     }
}
```

Note:
> BEARではどのクラスもコンストラクタの直後にonInject()が実行され、そのクラスが必要とする依存を整えます。（PHPUnitでのSetupメソッドのような働きをします）このクラスでは`BEAR_Query`サービスが`_query`プロパティにセットされます。これはDBオブジェクトを含んだSQLクエリーサービスオブジェクトです。

## コマンドラインからリソースの利用 ##

このリソースを利用してみましょう。
どこのディレクトリからでもかまいません、以下のコマンドを入力します。

```
$ bear read Post
```

以下のようなレスポンスが返ってくるはずです。

```
code
200
header
n/a
body
array (
  0 => 
  array (
    'id' => '1',
    'title' => 'タイトル',
    'body' => 'これは、記事の本文です。',
    'created' => '2011-07-01 22:30:25',
    'modified' => NULL,
  ),
  1 => 
  array (
    'id' => '2',
    'title' => 'またタイトル',
    'body' => 'そこに本文が続きます。',
    'created' => '2011-07-01 22:30:25',
    'modified' => NULL,
  ),
  2 => 
  array (
    'id' => '3',
    'title' => 'タイトルの逆襲',
    'body' => 'こりゃ本当に面白そう！うそ。',
    'created' => '2011-07-01 22:30:27',
    'modified' => NULL,
  ),
)
```
code, header, bodyとそれぞれの値が表示されました。
onRead内ではSQLクエリーで得られたarrayをreturnしているだけなのに、なぜこの３種類の情報が返って来ているのでしょうか？

## HTTPモデル ##

例えば以下のPHPスクリプトがweb公開エリアに置いてある事を考えてみてください。
```
<?php echo 'hello world'; ?>
```
PHPでは文字列をechoしているだけですが、実際のHTTPレスポンスは
```
HTTP/1.1 200 OK
Date: Fri, 01 Jul 2011 15:00:34 GMT
Server: Apache/2.2.19 (Unix) mod_ssl/2.2.19 OpenSSL/1.0.0d DAV/2 PHP/5.3.6
Content-Length: 11
Connection: close
Content-Type: text/html

hello world
```
とcode(=200), header, body(=hello world)の3種の情報が返っています。これと同じように考えてみてください。つまりbody部分だけを返すとcodeにリクエストの結果コード、headerにはメタ情報が付加されたリソースオブジェクトが返ります。

## フォーマットオプション ##

コマンドラインではフォーマットの指定ができます。

```
bear read Post -f table
```

```
+----+---------+----------------+---------------------+----------+
| id | title   | body           | created             | modified |
+----+---------+----------------+---------------------+----------+
| 1  | タイトル    | これは、記事の本文です。   | 2011-07-01 22:30:25 |          |
| 2  | またタイトル  | そこに本文が続きます。    | 2011-07-01 22:30:25 |          |
| 3  | タイトルの逆襲 | こりゃ本当に面白そう！うそ。 | 2011-07-01 22:30:27 |          |
+----+---------+----------------+---------------------+----------+
```

※日本語で罫線が残念な事になってますが...

CSVもつくれます。

```
$ bear read Post -f csv > post.csv
$ cat post.csv
```

```
1,タイトル,これは、記事の本文です。,"2011-07-01 22:30:25",
2,またタイトル,そこに本文が続きます。,"2011-07-01 22:30:25",
3,タイトルの逆襲,こりゃ本当に面白そう！うそ。,"2011-07-01 22:30:27",
```

Note:

> リソースはリソース状態（=リソースの本質的値）を持ちますがリソース表現を与えられクライアントに返されます。

## 関連項目 ##
  * [resource](resource.md)
  * [CLI](CLI.md)
  * [db](db.md)
  * [PEARマニュアル MDB2](http://pear.php.net/manual/ja/package.database.mdb2.php)