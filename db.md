# 導入 #

BEARは独自のデータベースクラスライブラリを持ちません。

汎用のライブラリを選択して利用しますが、[PEAR::MDB2](http://pear.php.net/manual/ja/package.database.mdb2.php) をBEARで使いやすいようにBEAR\_Mdb2クラスにDBページャーソートの機能を簡単に実現できる専用のクエリーツールクラス`BEAR_Query`が用意されています。

## BEAR\_Query ##

### config ###

| キー| 型 | 意味　| 必須？ |
|:---|:---|:---|:----|
| db | MDB2 | MDB2オブジェクト | Y   |
| ro | BEAR\_Ro | リソースオブジェクト  | Y   |
| table | string | テーブル名 | Y   |
| pager | int  |  ページング※1 | Y   |
| perPage | int | ページ毎のアイテム数 |     |
| offset | int | オフセット |     |
| options | array | DBページャーリンクオプション※2 |     |
| deleted\_at | bool | 論理削除対応 |     |
| sort | array | ソートオプション※3 |     |


  * ※1 perPageが０より大きいとき機能します。0だとperPageで指定したLIMIT文、1だとDBページングでselectされます。
  * ※2 ページャーリンクのオプションです。[PEAR::Pager](http://pear.php.net/manual/ja/package.html.pager.factory.php)と同じです。
  * `WHERE deleted_at IS NULL`を付加し、論理削除に対応します。
  * ※3 URLのクエリーに対応してDBでソートを行います。詳細は以下

### DBページャーとソートオプション ###

pager(DBページャー）とsortオプションはページやリソースでの記述なしに、クエリーに応じてリソース表現がかわるオプションです。

pager = 1, perPage > 0オプションではDBページングが行われ、ページャーリンクが{$pager.links}、ページャー情報が{$pager.info}にアサインされます。

sortオプションはソート可能なキー名を指定するとそのキーに応じたクエリーが行われる機能です。複数指定もでき、URLではカンマでくぎります。降順にするにはキーの前にマイナスをつけます。

URL例
```
?_sort=id,-date,age&type=bear
```

キーはDBカラム名そのままか、カラム名を表に出したくない時に代替のキーを指定します。

$id = array($privateColumn, $publicGet, $dir);

などと３つの配列で指定します。

| $privateColumn | string | DBカラム名 ||
|:---------------|:-------|:-------|:---|
| $publicGet     | string | `$_GET`クエリーキー　||
| $dir           | string | 降順昇順   | '+'または'-' |

```
    $id = array('id', 'id', '+');
    $date = array('created_at', 'date', '-');
    $this->_queryConfig['sort'] = array($id, $date); // ソート
```

`$_GET`クエリーに$publicGetが存在すればその方向でソートされます。-クエリーキーの前にがついていれば降順です。

```
?_sort=-id
```

複数のソートはカンマで区切ります。

```
?_sort=id,-date
```

DBのカラム名と違うキーでソートを行う場合には$publicGetKeyにcolumnNameと違う名前をつけます。$dirは+は昇順、-は降順です。

## PEAR::MDB2以外の利用 ##

PEAR::MDB2利用は必須ではありません、パフォーマンスや要求に応じてライブラリを選択できます。※他のほとんどのBEARのクラスと同様にDBクラスとBEARは密結合していません。

PEAR系
  * [MDB2](http://pear.php.net/manual/ja/package.database.mdb2.php)
  * [MDB\_QueryTool](http://pear.php.net/manual/ja/package.database.mdb-querytool.php)
  * [DB\_DataObject](http://pear.php.net/manual/ja/package.database.db-dataobject.php)
PHP純正
  * [PDO](http://jp.php.net/manual/ja/book.pdo.php)
  * [PHP::mysqli](http://jp.php.net/manual/ja/book.mysqli.php)
その他
  * [Zend\_Db](http://framework.zend.com/manual/ja/zend.db.html)
  * [docotrin](http://www.doctrine-project.org/)
  * [ADOdb](http://adodb.sourceforge.net/)


# 詳細 #

## 命名規則 ##

BEARはデータベースに特別なコラム名を利用することはありませんが、標準的な規則を以下に示します。

| id | int | プライマリーキー |
|:---|:----|:---------|
| created\_at | datetime | 作成日時     |
| updated\_at | datetime | 変更日時     |
| deleted\_at | datetime | 削除日時     |
| is\_XXX | tinyint | bool型    |

  * idはint型
  * 作成日時はcreated\_atフィールドにdatetime型
  * 更新日時はupdated\_atフィールドにdatetime型
  * 有効無効は削除日時とかねてdeleted\_atフィールドにdatetime型

論理削除は以下のように削除日時がNULLかどうか調べます。
```
WHERE users.deleted_at IS NULL
```
予約削除は以下のようにします。
```
WHERE users.deleted_at IS NULL OR users.deleted_at > '2009-08-01 00:00:00'
```

現在日時は`_BEAR_DATETIME_`がそのまま使えます。
リクエストで時間を合わせることができます。

## リンク ##

  * [bear-demo DBセレクト](http://code.google.com/p/bear-demo/wiki/select)