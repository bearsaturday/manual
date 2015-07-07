# 導入 #

OSXにcassandraとthriftをインストールします。

# 詳細 #

## cassandra ##

cassandraはmacportでインストールできます

```
$ sudo port install cassandra
```


## thrift ##

thriftを/usr/local/thriftにインストールする例です。2010年10月21日現在macportsで用意されていないようなのでソースからインストールします。

php-develが必要です。
phpがインストールされていればdeactivateしてからphp-develをインストールします
```
$ sudo port deactivate php
$ sudo port deactivate php-devel
```

boostというパッケージをmacportsでインストールしてからソースインストールします。
```
$ sudo install boost
$ svn co http://svn.apache.org/repos/asf/incubator/thrift/trunk /usr/local/thrift
$ ./configure --with-boost=/opt/local --with-libevent=/opt/local --prefix=/opt/local
$ make
$ sudo make install
```

PHP extentionのインストール
```
$ cd /usr/local/thrift/lib/php/src/ext/thrift_protocol/
$ ./configure
$ make
$ sudo make install
$ sudo cp /usr/local/thrift/lib/php/thrift_protocol.ini /opt/local/var/db/php5/
```

確認

```
$ php -i | grep 'thrift'
/opt/local/var/db/php5/thrift_protocol.ini,
thrift_protocol
PWD => /usr/local/thrift/lib/php
OLDPWD => /usr/local/thrift/lib/php/src
_SERVER["PWD"] => /usr/local/thrift/lib/php
_SERVER["OLDPWD"] => /usr/local/thrift/lib/php/src
```

apacheの再起動
```
$ sudo /opt/local/apache2/bin/apachectl restart
```