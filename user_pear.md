# Introduction #

グローバルのPEARの影響を受けないユーザー環境のPEARを構築します。インストールしたPHPにも影響を受ける事がありません。

ユーザー環境のpearは主にプロジェクトを横断するphpunitやbearコマンド等のbinツール(phpunit, dbunit, invoker, phpdepend, phpmd, phpcs, apigen, phpdox, docblox)に使います。

## グローバル環境のPEAR ##

PHPに付属するグローバルのPEARか、無ければ[go-pear](http://pear.php.net/manual/ja/installation.getting.php)でインストールします。

## ユーザー環境のPEAR ##

下記の手順でホームディレクトリ下にPEARフォルダ(~/pear)が作成され、ライブラリやコマンドがユーザー環境で使用できます。

pearフォルダと.pearrcを作成
```
$ pear config-create ~ ~/.pearrc
```

ユーザー環境のPEARをインストール
```
$ pear install -o PEAR
```

.bash\_profileにパスを追加

```
export PATH="$HOME/pear:$PATH"
export PHP_PEAR_INSTALL_DIR="$HOME/pear/php"
```

パスを有効に
```
$ . ~/.bash_profile 
```
PEARのパスとENV確認
```
$ which pear
{home}/pear/pear

$ env | grep PEAR
PHP_PEAR_PHP_BIN={home}/pear
PHP_PEAR_INSTALL_DIR={home}/pear/php
```

~/.pearcがあるのでpearコマンドの操作がユーザー環境のPEARに対する操作になります。

```
$ pear config-set auto_discover 1
$ pear install pear.phpunit.de/PHPUnit
```

pearでインストールしたコマンドもユーザー環境にインストールされます。

```
$ which phpunit
{home}/pear/phpunit
```

## プロジェクト環境のPEAR ##

プロジェクトフォルダにpearフォルダと.pearrcを作成します。

```
$ pear config-create /path/to/project /path/to/project/.pearrc
```
`-c`オプションで指定して操作します。
```
$ pear -c /path/to/project/.pearrc config-show
$ pear -c /path/to/project/.pearrc config-set auto_discover 1

$ pear -c /path/to/project/.pearrc install Cache_Lite
$ pear -c /path/to/project/.pearrc install pear.bear-project.net/bear-beta
```

### include path ###

プロジェクトのinclude\_pathは`/path/to/project/pear`になります。