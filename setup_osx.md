# 導入 #
OSXにApache2/PHP5の開発環境をmacportsでセットアップする一例です。
apacheの再起動なしでバーチャルドメインのサイトの設定を実現します。
純正でのセットアップはこちら、[OSX純正PHP環境](setup_osx_php.md)で


# 概要 #

  * OSX標準環境は使わずにMacportsでセットアップ
  * PHPの機能拡張（iMagickやCairo等グラフィックライブラリ、memcahceキャッシュ,spycYAMLパーサー）、および各サービスのインストール
  * バーチャンルホストを使用してhostファイルのみの変更で各プロジェクトに個別にルート(/)からアクセスできる。
  * MySQLはUTF-8で構築
  * webフォルダのパスは/var/www/<プロジェクト>/htdocs　localhostは/var/www/localhost/htdocs/
  * キャッシュやPEARのパッケージがwebから管理できる。
  * バージョンはPHP5.3, MySQL5.0,Apache2.2

# OSX標準 VS Macports PHP環境 #

> OSX10.5標準のPHP環境と比較。
メリット:
  * 画像ライブラリなど外部機能拡張のインストールにパッケージが使える
  * PEAR環境が標準的にある
  * PHPのコンパイルオプションが変更できる。
デメリット:
  * インストールに手間と時間がかかる
  * インストール時にエラー発生の可能性がパッケージより高い。

# 詳細 #

## Xcodeのインストール ##

macportsを利用するためにはOSX純正開発環境Xcodeが必要。http://developer.apple.com/technology/xcode.html から最新版Xcodeをダウンロード（要登録）して標準インストール。OSX付属のものよりバージョンが新しい。

## macportsインストール ##

macportsはバイナリまたはソースでインストールできる。

### バイナリインストール ###
http://www.macports.org/install.phpからダウンロードインストール
### ソースインストール ###
ホームの/local/srcに展開してコンパイル、インストールの例

```
$ mkdir -p ~/local/src
$ cd ~/local/src
$ svn co http://svn.macports.org/repository/macports/trunk/base macports
$ cd macports
$ ./configure
$ make
$ sudo mkdir -p /opt/local/share/macports
$ sudo make install
```

### portコマンドでパスが通ってなければパスを通す ###

新規ターミナルで下記が実行できるなら問題なし
```
 $ port help
```
パスが通ってないなら~/.bash\_profileを編集
```
export PATH="/opt/local/bin:/opt/local/sbin:$PATH"
export MANPATH="/opt/local/share/man:$MANPATH"
```
ターミナルを新規に開いてport helpを試す

### macports更新 ###

```
$ sudo port -v selfupdate
$ sudo port sync
```


### PHP5+Mysql+画像ライブラリのインストール ###

**MySQL5.1の場合**

```
$ sudo port install mysql5-devel mysql5-server-devel
```

**MySQL5.0の場合
```
$ sudo port install mysql5 mysql5-server
```**


ライブラリ（GD、iMagick, Cairo, キャッシュ、YAML)とphpのインストール
```
$ sudo port install memcached syck ImageMagick +jpeg2+mpeg+perl+q32+no_x11 GraphicsMagick cairo +no_x11
$ sudo port install php5 +apache2+pear
$ cd /opt/local/apache2/modules
$ sudo /opt/local/apache2/bin/apxs -a -e -n "php5" libphp5.so
```


### php拡張のインストール ###
```
sudo port list php5-*
```
でインストール可能なPHP5拡張パッケージを確認

拡張インストール例
```
sudo port install php5-apc php5-exif php5-ftp php5-gd php5-http php5-iconv php5-imagick php5-mbstring php5-mcrypt php5-memcache php5-mysql php5-openssl php5-pcntl php5-readline php5-sockets php5-sqlite3 php5-syck php5-tidy php5-uploadprogress  php5-zip
```
デバッガー
```
sudo port install php5-xdebug
```

### 起動スクリプトをロード ###
-wで起動時もロードされる
```
$ sudo launchctl load -w /Library/LaunchDaemons/org.macports.mysql5.plist
$ sudo launchctl load -w /Library/LaunchDaemons/org.macports.apache2.plist
$ sudo launchctl load -w /Library/LaunchDaemons/org.macports.memcached.plist
```

起動にエラーがあっても表示されないことがあるので注意


  * php5/ImageMagickはで必要な拡張をvariantsで把握して指定します。 例 $ port variants php5
  * コンパイルオプションを変更したいときはeditで編集します。 例$port edit php5
> " 以前mysqlのvariantsだったserverは独立したパッケージになりました。PHPの拡張もpeclよりphp5-**パッケージの方が安定しています**

### トラブルシューティング ###

コンパイルでエラーが出る場合がある。エラーの原因により対処法は違うが以下の対処で解決する場合がある。

  * cleanコマンドを使ってみる 例）$sudo port clean php5
  * ソースインストールで問題があるならdmg版で再インストールしてみる ※1
  * variantns（+ipcなど）を減らしてみる
  * sudo port edit php5などとしてconfigureを変更してみる
  * ソースのチェックサムエラーが出ることがある。
```
Error: Checksum (md5) mismatch for php-5.3.0.tar.bz2
Error: Checksum (sha1) mismatch for php-5.3.0.tar.bz2
Error: Checksum (rmd160) mismatch for php-5.3.0.tar.bz2
Error: Target org.macports.checksum returned: Unable to verify file checksums
Error: Status 1 encountered during processing.
```
この場合はネットからphp-5.3.0.tar.bz2を探してソースを置き換える。
ソースディレクトリはSpotlightで検索。php5は（/opt/local/var/macports/distfiles/php5）

### configureオプションを変更 ###
例）php5に　-enable-memory-limitをつける。※実際はこのオプションは5.2.1から常にonになっているで不要

```
$ sudo port edit php5
```
myvar variantとして以下を追記
```
variant myvar {
    configure.args-append \
        --enable-memory-limit
}

```
上記でつくったものは+myvarバリアントでインストールできる。

### 確認 ###

http://localhost/ でIt worksが出るのを確認。
アンコメントして以下の行を有効化。httpd-vhosts.confはmy−httpd-vhosts.confにする。mod\_php.confも追加。

※バックアップをとって、オリジナルを編集しないでオリジナルをバックアップにしてmyとかつけたファイル名を使用しているのはmacportでupdateしたときにファイルが上書きして消されないため（以前updateで全部消えた）


> /opt/local/apache2/conf/httpd.confを編集し下記を検索してアンコメント。

```
Include conf/extra/httpd-autoindex.conf
Include conf/extra/httpd-info.conf
Include conf/extra/my-httpd-vhosts.conf
```

phpの読み込みをファイルの最後など適当な場所に追記。

```
Include /opt/local/apache2/conf/extra/mod_php.conf
```

### バーチャルホスト設定 ###

```
$ sudo cp /opt/local/apache2/conf/extra/httpd-vhosts.conf /opt/local/apache2/conf/extra/my-httpd-vhosts.conf
```

/opt/local/apache2/conf/extra/my-httpd-vhosts.confを編集。既存のものを消去して下記のものに置き換える。

```

#
# Use name-based virtual hosting.
#
NameVirtualHost *:80

<Directory /var/www>
  AllowOverride All
  Options Indexes FollowSymLinks ExecCGI MultiViews
  order deny,allow
  Allow from all
  DirectoryIndex index.html index.htm index.php
</Directory>
        
<VirtualHost *:80>
  ServerName localhost
  VirtualDocumentRoot /var/www/%0/htdocs
</VirtualHost>

```

## /etc/hostsファイル編集 ##

<プロジェクト>をlocalhostに向ける為にhostsファイルを編集する.

> 例) <プロジェクト>がapp.sample1 -4 の４つのプロジェクトの場合
```
$ sudo vi /etc/hosts
```
```
127.0.0.1    localhost 
127.0.0.1    app.sample1 app.sample2 app.sample3 app.sample4
```
※スペース区切りで追加
※hostsファイルの変更後にapacheリスタート等は不要

/var/www/<プロジェクト>/htdocs/index.htmlがhttp://<プロジェクト>/index.htmlでアクセスできる。
できなければ ping app.sample1等として127.0.0.1を向いているか確認。

Apacheスタート

```
$ sudo /opt/local/apache2/bin/apachectl configtest
Syntax OK
$ sudo /opt/local/apache2/bin/apachectl restart
```

extentionのインストール。cairoが不要ならここはスキップ

```
$ sudo pecl config-set preferred_state beta
$ sudo pecl install cairo_wrapper
```

トラブルシューティング

imagickでMagickWand-configがないとエラーが出たらシンボリックリンクを張り再度 pecl install imagick

```
sudo ln -s /opt/local/bin/MagickWand-config /usr/bin/
```

php.iniを編集

> /opt/local/etc/php.ini

apcを開発のため一時的に外す。（パースエラーで"真っ白"がある場合がある）
パーケージでいれたエクステンションは/opt/local/var/db/以下のファイルで設定する。


### php.iniをweb用とCLI用と２種類用意 ###
web(mod\_php)用php.ini

```
$ sudo cp /etc/php.ini.default /opt/local/etc/php.ini
```

CLI用php.ini
```
$ sudo cp /etc/php.ini.default /opt/local/etc/php-cli.ini
```


```
$ vi /opt/local/var/db/php5/apc.ini 

;extension=apc.so
```


以下のタイムゾーンを設定

```
date.timezone = Asia/Tokyo
```

以下を追記

```
extension=cairo_wrapper.so
[Zend] 
zend_extension=/usr/lib/php/extensions/no-debug-non-zts-20060613/ZendDebugger.so
zend_debugger.allow_hosts = 127.0.0.1
zend_debugger.expose_remotely = always
```

PEAR使用など完全にPHP5専用に以降してない場合などエラーの出力レベル感度を下げる場合
[{{
error\_reporting = E\_ALL | E\_STRICT
}}
を
[{{
error\_reporting = E\_ALL
}}
に変更。
PHP5.3でE\_DEPRECATEDを抑制する場合は
```
error_reporting = E_ALL & ~E_USER_DEPRECATED
```

## MYSQL設定 ##
初期化
```
$ sudo -u mysql mysql_install_db5
```
#### 再起動 ####
```
$ sudo /opt/local/share/mysql5/mysql/mysql.server start
```

### パスワード設定 ###
```
/opt/local/lib/mysql5/bin/mysqladmin -u root password 'new-password'
/opt/local/lib/mysql5/bin/mysqladmin -u root -h localhost password 'new-password'
```

### 実行してみる ###
```
cd /opt/local ; sudo /opt/local/lib/mysql5/bin/mysqld_safe &

$ mysql5 -u root -p
```

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.1.40-log Source distribution

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> exit
Bye
```

### my.cnfの作成と設定 ###

```
$ sudo cp /opt/local/share/mysql5/mysql/my-small.cnf /etc/my.cnf
```

デフォルトキャラクタの設定­をmy.confに追記。console.appでログが確認しやすいように/var/log/opt/にクエリーログを保存設定

/etc/my.cnf
```
[client]
default-character-set = utf8

[mysqld]
default-character-set = utf8
skip-character-set-client-handshake
log=/opt/local/var/db/mysql5/query.log
long_query_time = 5
log-queries-not-using-indexes
log-slow-queries=/opt/local/var/db/mysql5/query-slow.log

[mysqldump]
default-character-set = utf8
```

※dataディレクトリを変更したい場合なら以下も[mysqld](mysqld.md)に追記。書き込み権限も要設定。設定しないと/opt/以下の標準パスにDBデータが保存される。

```
 datadir=/Users/koriyama/Documents/mysql/Data­/mysql5
```

### MySQL再起動 ###
```
$ sudo /opt/local/share/mysql5/mysql/mysql.server restart
```


## ソケットファイルのパスの設定 ##

mysqlのsocketファイルのパスを確認

```
$ mysql_config5 | grep socket
        --socket         [/opt/local/var/run/mysql5/mysqld.sock]
```

opt/local/etc/php5/php.ini（または/opt/local/etc/php.ini )を編集


```
pdo_mysql.default_socket= /opt/local/var/run/mysql5/mysqld.sock
mysql.default_socket = /opt/local/var/run/mysql5/mysqld.sock
mysqli.default_socket = /opt/local/var/run/mysql5/mysqld.sock
```


### 確認 ###
DB文字コード

```
$ mysql5 -u root

> show variables like 'char%';
+--------------------------+-----------------------------------------+
| Variable_name            | Value                                   |
+--------------------------+-----------------------------------------+
| character_set_client     | utf8                                    |
| character_set_connection | utf8                                    |
| character_set_database   | utf8                                    |
| character_set_filesystem | binary                                  |
| character_set_results    | utf8                                    |
| character_set_server     | utf8                                    |
| character_set_system     | utf8                                    |
| character_sets_dir       | /opt/local/share/mysql5/mysql/charsets/ |
+--------------------------+-----------------------------------------+


```
となればOK

### PHPからのlocalhostソケット接続 ###
> パスワード設定している場合は""を"password"に変更

```
$ php -r 'mysql_connect(":/opt/local/var/run/mysql5/mysqld.sock", "root", "");'; 
$ php -r 'new mysqli("localhost", "root", "", "mysql");'
$ php -r 'new PDO("mysql:host=localhost;dbname=mysql", "root", "");'
```

これでエラーが出なければOK

### トラブルシューティング ###


. ERROR! Manager of pid-file quit without updating file.というエラーがコンソールに出てmysqlがスタートできない事があるが原因がコンソールに表示されない。

エラー内容をtailする
```
sudo tail -f  /opt/local/var/db/mysql5/マシン名.err
```

例えば
```
[ERROR] Fatal error: Can't open and lock privilege tables: Table 'mysql.host' doesn't exist
091019 12:50:26  mysqld ended
```
これはmysqlテーブルが読めない（ない）場合のエラー

これで対処
```
sudo -u mysql mysql_install_db5
```

my.cnfに誤りがあり指定しているディレクトリやファイルをがない場合にエラーが出る事がある。パスをチェック。
またはmysqlのプロセスが残ってるときがある。psでみつけてkillで削除

```
 $ ps ax | grep mysql
```
```
 $ kill [pid]
```


どうしてもうまくいかない場合はuninstall, installをやり直してみる。
```
sudo port uninstall mysql5
sudo rm -rf /opt/local/var/db/mysql5
sudo port install mysql5 mysql5-server
（または）
sudo port install mysql5-devel mysql5-devel-server

```

/opt/local/var/run/mysql5/mysqld.sockのエラーの場合

ソケットファイル作成で対処。

```
$ sudo mkdir /opt/local/var/run/mysql5
$ sudo chown mysql:mysql /opt/local/var/run/mysql5/
$ sudo chmod 775 /opt/local/var/run/mysql5
```

```
Fatal error: Class 'mysqli' not found in Command line code on line 1
```
このエラーだとphp5-mysqlがインストールされてない


## Webアプリ管理画面 ##

### apc, memcaheの管理画面を用意 ###

```
$ sudo cp /usr/share/php/apc.php /var/www/localhost/htdocs/
$ sudo cp /usr/share/php/docs/memcache/memcache.php /var/www/localhost/htdocs/
```

memcaheの管理画面でID/passとmemcacheサーバーをlocalhostに設定

> /var/www/localhost/htdocs/memcache.php

```
define('ADMIN_USERNAME','memcache');    // Admin Username
define('ADMIN_PASSWORD','password');    // Admin Password
$MEMCACHE_SERVERS[] = 'localhost:11211'; // add more as an array
//$MEMCACHE_SERVERS[] = 'mymemcache-server2:11211'; // add more as an array

```
mysqlのクエリーログをwebから見られるように権限変更

```
$ sudo chmod 664 /opt/local/var/db/mysql5/query.log
$ sudo chmod 664 /opt/local/var/db/mysql5/query-slow.log
```

### PEARのフロント画面 ###
[PEAR\_Frontend\_Web](http://pear.php.net/package/PEAR_Frontend_Web/docs)をインストールしてPEARのフロント画面を用意。

```
$ sudo pear install -a PEAR_Frontend_Web
$ sudo cp -r /usr/share/php/docs/PEAR_Frontend_Web/docs/ /var/www/localhost/htdocs/pear
$ cd /var/www/localhost/htdocs/pear/
$ sudo mv index.php.txt index.php
```
確認

```
$ http://localhost/pear/
```

**トラブルシュート**

```
Warning: Can not find config file, please specify the $pear_user_config variable in /pear/index.php
```

というエラーが出たらwebから書き込めるディレクトリを用意してpear.confを指定。

pear config-createで作成するとpearが最後に自動的にきて面倒なので、存在しないconfを指定して一度目のアクセスで作ってもらいます。リロードすると正しく表示される。

```
$ cd $ mkdir local $ chmod 777 local
$ sudo pear config-create /usr/share/php ~/.pearrc
$ sudo vi  index.php
$ pear_user_config = '/Users/<ユーザー名>/local/pear.conf';
```

などとする。また
```
//$pear_frontweb_protected = true;
```

とアンコメントすると認証をかけてない警告を抑制できる。

webでインストールするためのパーミッション設定。

```
$ sudo chown -R www /usr/share/php
```

## linuxとパスを揃える ##
ライブサーバーでの運用を考えPEARのパスを揃えりためシンボリックリンクを張ります。

```
$ sudo ln -s /opt/local/lib/php /usr/share/php
$ sudo ln -s /opt/local/lib/php /usr/share/pear
```
上記がUbuntu/Debian系下記がRedHat系です。どちらも張っといていいと思います。


# 付録 #

## PHP\_CodeSnifferを使う場合 ##

PHPのコードチェックをする[http://pear.php.net/manual/ja/package.php.php-codesniffer.php](PHP_CodeSniffer.md)はコードの整形のチェックだけでなくZendCodeAnalyzerを使うと潜在的なバグの可能性まで指摘までしてくれる（未使用変数の放置、届かないコードなど）。
```
sudo  phpcs --config-set zend_ca_path '/Applications/Zend/Zend\ Studio\ for\ Eclipse\ -\ 6.1.0/plugins/com.zend.php.codeanalyzer.macosx_6.1.0.v20080907/resources/ZendCodeAnalyzer'
```
  * [PHP\_CodeSniffer](http://pear.php.net/manual/ja/package.php.php-codesniffer.php)
  * [miau's blog? PHP\_CodeSniffer の --standard オプション比較](http://miau.s9.xrea.com/blog/index.php?itemid=914)
  * [Eclipseから実行](http://homepage3.nifty.com/renoiv/php/phpcs/eclipse.html)

## Eclipse - PDTインストール ##

[PDT](http://www.eclipse.org/pdt/)
[日本語化パック](http://www.igapyon.jp/blanco/nlpack/eclipse/pdt.html)

PDTをダウンロード展開して、日本語化パックファイルを該当フォルダ(plugins, features)にコピーして起動。

### SVN ###
[subversive](http://www.eclipse.org/subversive/downloads.php)を参照。

eclipse > ヘルプ > Install New Software > AddでSVN用更新サイトを追加

  * Name : subversive
  * URL: http://download.eclipse.org/technology/subversive/0.7/update-site/

### Eclipse設定 ###

割当メモリを増やす
```
$ vi <インストールディレクトリ>/eclipse/Eclipse.app/Contents/MacOS/eclipse.ini
```
```
-Xms512m
-Xmx512m
```
※1G以上はうまくいかないみたい

設定をかえたら初期化してクリーンスタート
```
$ cd <インストールディレクトリ>
$ eclipse/Eclipse.app/Contents/MacOS/eclipse -initialize
$ eclipse/Eclipse.app/Contents/MacOS/eclipse -clean &
```

## Graphviz ##
macportsになく、公式サイトのパッケージで

http://www.graphviz.org/Download_macos.php

# Eclipse #
Eclipse + PDTのセットアップは [Eclipseセットアップ](http://code.google.com/p/bear-project/wiki/eclipse)で

## PDTでdebugするために ##
iniファイルはディレクトリに入ったので読んでくれないのでphp.iniにエクステンションをまとめて記述。

```
extension=apc.so
extension=exif.so
extension=ftp.so
extension=gd.so
extension=http.so
extension=iconv.so
extension=imagick.so
extension=mcrypt.so
extension=memcache.so
extension=mysql.so
extension=mysqli.so
extension=pdo_mysql.so
extension=openssl.so
extension=pcntl.so
extension=posix.so
extension=sockets.so
extension=sqlite.so
extension=sqlite3.so
extension=pdo_sqlite.so
xtension=syck.so
extension=tidy.so
extension=uploadprogress.so
zend_extension=/opt/local/lib/php/extensions/no-debug-non-zts-20090626/xdebug.so
xdebug.remote_enable=On
xdebug.remote_host=localhost
xdebug.remote_port=9000
xdebug.remote_handler=dbgp
extension=xsl.so
extension=zip.so
```
> /opt/local/var/db/php5/内のファイルの読み込みをコメントにします。

### Zend Debugger ###

Zend Studioを使う場合は以下のZendDebuggerをインストールします。XDebugとの共存はできません。

http://downloads.zend.com/pdt/server-debugger/ からダウンロードしてコピー、デバックするプロジェクトのhtdocsにdummy.phpが必要なのでこれもコピー

```
$ sudo cp ~/Downloads/ZendDebugger-5.2.14-darwin8.6-uni/5_2_x_comp/ZendDebugger.so /usr/lib/php/extensions/no-debug-non-zts-20060613
$ cp ~/Downloads/ZendDebugger-5.2.14-darwin8.6-uni/dummy.php /var/www/localhost/htdocs
```



# メンテナンス #
更新されたパッケージを新しくし、古いバージョンや使われなくなったパッケージを削除する為に以下を時々実行します。

Note:
> パッケージが多いとアップグレードにかなりの時間がかかります。

```
 $ sudo port sync
 $ sudo port selfupdate
 $ sudo port clean --dist outdated
 $ sudo port upgrade outdated
 $ sudo port -u uninstall
```