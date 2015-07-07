# Introduction #

試用と開発用途のためBEARと拡張機能がセットアップされたUbuntu9.1 Serverのvmwareイメージが利用できます。※BEAR以外の利用にも便利だと思います。お使いください。

※以下の組み合わせで動作確認できています。(2010.3.31)

  * OSX + VM Fusion 3.0.2
  * XP + VMware Player 2.5.3, 3.0.1

# Details #

## ダウンロード ##
[Ubuntu for BEAR(bear.local) ダウンロード ](http://www.kumasystem.com/bear/vmware/Ubuntu4BEAR.vmwarevm.zip) (e3a536cb1b4609d6efd7e50c910db64d)


  1. ダウンロードしてVmwareで起動します。(Ubuntu4BEAR.vmx)
  1. 「移動」か「コピー」か訪ねられたら「移動」を選びます。
  1. アカウントbear / bearでログインして、/sbin/ifconfigでipを調べます（DHCPです）
  1. ホスト（自分のPC）のhostsファイルを編集します

※上記３で調べたゲストのipが172.16.38.129の場合

```
172.16.38.129 bear.local test.bear.local demo.bear.local
```

  1. http://bear.local/ をアクセスして確認します。サーバー管理ツールを利用したりサーバー情報が確認できます。
  1. http://demo.bear.local/ ではbear-demoと同じデモが試せます。

## アップデート ##

OSパッケージ/ PEARパッケージ /　ローカルサイトすべてを最新版にします。

```
$ sudo apt-get update
$ sudo apt-get ugrade
```

bear.locaサイト アップデート
```
$ cd /usr/local/bear/www/bear.local/ ; sudo svn update
```
pear アップデート
```
$ sudo pear channel-discover pear.zfcampus.org
$ sudo pear channel-discover pear.firephp.org
$ sudo pear upgrade-all
```
## アカウント情報 ##

  * 一般ユーザー bear / bear
  * rootユーザー  root / bear
  * mysqlユーザー root / bear

## 新サイトのセットアップ ##
※myapp.bear.localという新しいサイトの例
```
$ bear init-app ~/www/myapp.bear.local
```

hostsにmyapp.bear.localを追加

```
172.16.38.129 bear.local test.bear.local demo.bear.local myapp.bear.local
```
http:://myapp.bear.local/
で確認します。※apacheの再起動などは必要ありません。

## アクセス ##

### ファイル(Samba) ###
Sambaが利用できます。

  * OSXからはFinder > 移動 > サーバーに接続でsmb://bear.local/に接続します。
  * windowsからは¥¥bear.local¥bearでアクセスします。

### SSH ###
```
ssh bear@bear.local
```
### mysql ###
http://bear.local/ のweb管理ツールを使うか、3306が開いているのでホストからつなげます。

# サーバー構成 #

  * Ubunutu 9.1
  * PHP 5.2
  * MySLQ 5.1