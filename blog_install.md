## PEARインストール ##

PEARコマンドでダウンロード＆インストールします。
```
$ sudo pear channel-discover pear.bear-project.net
$ sudo pear install -a bear/BEAR-beta
```
確認します。
```
$ bear --version
```

アプリケーションをインストールします。
```
$ bear init-app /path/to/www/bearblog.local
```

Note:
> この例では/path/to/www/blog.local/htdocs/がvhostとしてbearblog.localでアクセスできると仮定しています。/path/to/www/をvhost設定に合わせて変更してください。

## 関連項目 ##

  * [Install](Install.md)
  * [バーチャルホスト設定](http://code.google.com/p/bear-project/wiki/setup_osx#%E3%83%90%E3%83%BC%E3%83%81%E3%83%A3%E3%83%AB%E3%83%9B%E3%82%B9%E3%83%88%E8%A8%AD%E5%AE%9A)