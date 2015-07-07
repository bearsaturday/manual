## MySQLにblogbearデータベースを作成¶ ##

次に、ブログで使用するデータベースをセットアップしましょう。今は、投稿記事を保存するためのテーブルをひとつ作成します。テスト用にいくつかの記事も入れておきましょう。次のSQLをデータベースで実行してください。

## blogbearデーターベースを作成 ##
```
CREATE DATABASE `blogbear` DEFAULT CHARACTER SET 'utf8';
```

## postsテーブルを作成 ##
```
CREATE TABLE posts (
id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
title VARCHAR(50),
body TEXT,
created DATETIME DEFAULT NULL,
modified DATETIME DEFAULT NULL
);
/* それから、テスト用に記事をいくつか入れておきます。 */
INSERT INTO posts (title,body,created)
VALUES ('タイトル', 'これは、記事の本文です。', NOW());
INSERT INTO posts (title,body,created)
VALUES ('またタイトル', 'そこに本文が続きます。', NOW());
INSERT INTO posts (title,body,created)
VALUES ('タイトルの逆襲', 'こりゃ本当に面白そう！うそ。', NOW());
```

## データベース接続設定 ##
bear init-appコマンドでインストールしたApp/app.ymlにDBの接続設定を[DSN](http://pear.php.net/manual/ja/package.database.mdb2.intro-dsn.php)で記します。

```
App_Db:
  dsn:
    default: 'mysql://root:@localhost/blogbear'
    slave  : 'mysql://root:@localhost/blogbear'
    test   : 'mysql://root:@localhost/blogbear'
```

Note:
> app.ymlは各クラスの生成ヒントが記されたファイルですが、ここでは各ライブラリが使う設定ファイルのように理解してもらっても構いません。

## テーブルスキーマの設定 ##

ORMを使用していないのでありません。