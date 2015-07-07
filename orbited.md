※未編集

# 導入 #

[Orbited](http://orbited.org/)はJavaScriptでソケット通信を可能にするソケットプロキシーサーバーです。webのリアルタイム通信が利用できます。
同種の技術に[エアリアル](http://d.hatena.ne.jp/viver/20080511/p1)がありますが、OrbitedはFlashが不要です。[ライブデモ](http://orbited.org/wiki/LiveHelp)参照。

# インストール #

  1. python
  1. twisted
  1. orbited
  1. python用JSONモジュール

とインストールします。
[Orbitedインストール](http://orbited.org/wiki/Installation)参照してください。web用のmod\_pythonのセットアップなどは不要です。


## OSX(10.5以降）にインストール ##

  1. pythonは付属のものでかまいません。
  1. http://twistedmatrix.com/trac/ からdmgでtwistedをインストール
  1. http://peak.telecommunity.com/dist/ez_setup.py をダウンロードして実行
```
$ python ez_setup.py setuptools
$ easy_install orbited

```
  1. http://orbited.org/browser/trunk/daemon/orbited.cfg をダウンロードして/etc/orbited.cfgとして設置

例）
```
$ wget http://peak.telecommunity.com/dist/ez_setup.py
$ python ez_setup.py setuptools
$ sudo python ez_setup.py setuptools
$ sudo easy_install orbited
$ wget http://orbited.org/browser/trunk/daemon/orbited.cfg
$ wget http://orbited.org/export/675/trunk/daemon/orbited.cfg
$ mv orbited.cfg /etc/
$ easy_install python-cjson
```


## コンフィグファイル ##

インストーラーでは設置されないcfgファイルは自分で/etc/orbited.cfgに置く例です。[access](access.md)に許可するホストのアクセスを必要なだけかきます。クライアントは`*`とかけますががサーバーは`*`とはかけません。以下はexcite/googleのweb、それにIRCに接続を許可した例です。

例
```
* -> www.excite.co.jp:80
* -> www.google.co.jp:80
* -> irc.freenode.net:6667
```

## 起動 ##

通常起動とデーモンでの起動がある（らしい）です。インストールでパスは通ってるが起動したディレクトリにログができるので注意。ログが表示され、cfgでのアクセス権が不十分だとエラーが表示されます。

ヘルプ
```
$ orbited -h
```
開発用起動
```
$ orbited --profile --config /etc/orbited-debug.cfg
```


### トラブルシューティング ###
```
01/19/10 10:31:27:756 ERROR  orbited.start	Aborting; Unknown user or group: 'getpwnam(): name not found: orbited'
```
これで終了した場合はコンフィグファイルでuserを書き換えます


### アクセス ###
http://localhost:8000/static/ で確認。上記コンフィグでtelnetやIRCのdemoページが確認できます。