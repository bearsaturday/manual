# 導入 #

OSX純正のPHP環境をセットアップする一例です。
簡単ですがMac Portsのインストール例と違いiMagick/memchaced/Cairo/syckなどのエクステンションはインストールされません。

# Details #

## httpd.conf ##

/private/etc/apache2/httpd.confを編集

#LoadModule php5\_module        libexec/apache2/libphp5.so

の# を消してシステム環境設定＞Web共有を停止＆共有

### PEARのインストール ###

PHPが--without-pearでコンパイルされいているので手動でインストールが必要。

まずはデフォルトの場所(/private/etc/apache2/extra)にインストールします。

```
 $ curl http://pear.php.net/go-pear | sudo php
```
1-7, 'all' or Enter to continue: のところで
```
 1. Installation prefix ($prefix) : /private/etc/apache2/extra
 2. Temporary files directory     : $prefix/temp
 3. Binaries directory            : $prefix/bin
 4. PHP code directory ($php_dir) : $prefix/PEAR
 5. Documentation base directory  : $php_dir/docs
 6. Data base directory           : $php_dir/data
 7. Tests base directory          : $php_dir/tests
```
変更しないで続行、質問にはデフォルトで答えれば完了。

debian/ubuntu系の/usr/share/phpにも対応できるようにシンボリックリンクを張ります。
RedHat系なら/usr/sahre/pearです。

```
 $ sudo mkdir -p /usr/share/php
 $ sudo ln -s /private/etc/apache2/extra/PEAR /usr/share/php
```
pearのパスを通す
```
$ vi ~/.bash_profile
```
/usr/share/pear/binを追加
```
export PATH=/opt/local/bin:/opt/local/sbin:/usr/share/php/bin:$PATH
```
php.ini

```
$ sudo vi /etc/php.ini
```
```
;***** Added by go-pear
include_path="/usr/share/php:."
```
PEAR確認
```
$ pear version
$ pear config-show
```

### php.iniをweb用とCLI用と２種類用意 ###
web(mod\_php)用php.ini

```
$ sudo cp /etc/php.ini.default /etc/php.ini
```

CLI用php.ini

```
$ sudo cp /etc/php.ini.default /etc/php-cli.ini
```