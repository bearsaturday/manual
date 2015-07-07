# Introduction #

UbuntuにBEARと周辺ライブラリ、サンプルアプリを開発用にインストール。

# Details #


## taskselでインストール ##
```
 $ sudo tasksel
```

でBasic Server, LAMP Server, OpenSSH Server, Samba Serverを選択してインストール。詳細は以下を参照
  * http://blog.netblue.jp/2009/10/30/ubuntu-9-10-server/

## phpおよび拡張機能をインストール ##
```
sudo apt-get install php5 php5-dev php-pear php5-imagick php5-memcache php5-syck php5-xdebug php5-cli php5-gd php5-mysql php5-sqlite libgv-php5 graphviz
```

## vhost設定 ##
```
$ cd /etc/apache2/sites-available/
```
test.bearvmを作成
```
<VirtualHost *:80>
ServerName test.bear.vm
DocumentRoot /var/www/test.bear.vm/htdocs
</Virtualhost>
```

有効化
```
$ sudo apache2ctl restart
```