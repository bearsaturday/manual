# 導入 #

PHPのインラインコメントでユーザー作成のBEARアプリケーションのAPIドキュメントが簡単に作成できます。

# 作成 #
phpDocumentorを使用しています。標準の設定を変更するにはappdoc.iniを編集します。

```
$ cd `<アプリケーションルート>`/data/package/ 

$ phpdoc -c appdoc.ini

または

$ sudo chmod 755 make_doc.sh
$ ./make_doc.sh
```

# 結果 #

アプリケーションAPIドキュメントが`<アプリケーションルート>`/data/api/に作成されます。