# 導入 #

作成したアプリケーションはPEARパッケージにする事が簡単にできます。

# 詳細 #

data/package/make\_package.phpの内容を編集して依存パッケージやパッケージ作成者などの情報を入力しphp make\_package.phpとして出てくる説明に従います。

```
php make_package __uri
```

または

```
php make_package pearサーバー名
```

としてpackage.xmlをApp/に作成
```
$ cd ../../
$ pear pacakge-validate
$ pear pacakge
```
とすれば アプリケーションパッケージができあがります。他サーバーにこのアプリケーションをインストールするときは以下のようにします。
```
$ pear install -a BearApp-0-0-1.tgz //ローカルパッケージの場合
```
PEARの依存関係なども解決してアプリケーションがインストールされます。アプリケーションは\_pear