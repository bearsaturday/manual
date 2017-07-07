# This package is not maintained anymore and has been superseded. Use [BEAR.Sunday](https://bearsunday.github.io/) Instead.
# このパッケージはメンテナンスされていません。新規インストールは推奨されません。

BEAR.Saturdayが依存する[PEARのパッケージ](https://pear.php.net/packages.php)のいくつかもメンテナンスされていません。
 
 * Net_UserAgent_Mobile
 * XML_RSS
 * Console_Color
 * File_SearchReplace
 * Net_Server

----

## 導入 ##

BEARのインストール／アップデートはPEARコマンドを使って行います。

## 利用条件 ##

  * PHP5.2または5.3（BEAR自身はPHP5.2コード）
  * PEAR環境
  * Safeモード可
  * エクステンション（オプション）APC, GD, Cairo, syck, Xdebug, xhprof

# インストール方法 #
２つの方法があります。

  * PEARインストール
  * composerインストール

新規のプロジェクトはcomposerインストールを推奨します。

## composerインストール ##

composer installは[composer](http://getcomposer.org/)を使ってプロジェクト単位で依存PEARライブラリをインストールします。手順はBEARSaturday.Skeletonの[README](https://github.com/BEARSaturday/BEARSaturday.Skeleton)をご覧下さい。


## PEARインストール ##

```
$ sudo pear config-set auto_discover 1
$ sudo pear config-set preferred_state alpha
$ sudo pear channel-update pear.php.net
$ sudo pear channel-discover bearsaturday.github.io/pear
$ sudo pear upgrade pear
$ sudo pear upgrade-all
```

```
（開発）
$ sudo pear install -a bear/BEAR-beta
（運用）
$ sudo pear install bear/BEAR-beta
```

コマンドラインで確認

```
$ bear --version
bear version  x.x.x
```


# はじめてみよう #

## アプリケーションプロジェクト作成 ##
app.testというプロジェクトを作成します。プロジェクト名がフォルダの名前になります。

```
$ bear init-app /path/to/app.test
```

これでアプリケーションのセットアップは完了します。
.htaccessの設置と設定、デバック用画面のシンボリックリンク、tmp/logフォルダの書き込み権限などの設定も行われます。

※/path/to/app.test/htdocsがweb公開エリアという設定がapache等で必要です。

## 確認 ##

`http://app.test/HelloWorld.php`を確認します。

## ドキュメントルート以外への設置 ##
BEARはwebのドキュメントルート以外設置できますが、現在開発用画面はルートへのアクセスを前提としています。必要なのは（アンダースコア２つ）で始まるフォルダbear, panda, editとApp.phpをinclude\_pathとして認識させてあげることです。以下のどちらかの方法で対処できます。

  * ドキュメントルートにファイルを置かない方法
> > mod\_rewriteを使用します。TOMさんのブログをご覧ください　→　[Stellaqua - ＴＯＭの技術日記 BEARで始めるWebアプリケーション開発 その1](http://d.hatena.ne.jp/stellaqua/20100120/1263983424)
  * ドキュメントルートにファイルを一部置いてもいいときの方法
> > BEAR使用シンボリックリンクの移動します。アプリケーションの使用するパス（css/js/など）はApp/view/layout/内のレイアウトファイルで変更します。
```
 $  mv /path/to/app/htdocs/__bear /path/to/docroot
 $  mv /path/to/app/htdocs/__panda /path/to/docroot
 $  mv /path/to/app/htdocs/.htaccess /path/to/docroot
```

BEARがシステムとして使用するのは開発の時に使うで始まるフォルダだけです。BEARでは「URLを解析してコントローラやモデルを呼び出すような仕組み」はありません。基本的にはどこでも設置できます。


## トラブルシューティング ##

BEARはinclude\_pathの設定に.htaccessを利用してます。パスは自動で以下の３つに通っているはずですが `Class XXX not found`などのエラーが出た場合はこのパスを確認してください。

  * App.phpのあるディレクトリ　(bear init-appを行ったディレクトリです）
  * PEARディレクトリ (pear config-showのphp\_dirです）
  * PEARディレクトリ/BEAR/vendors/PEAR/



# その他 #

### `FireFoxエクステンション` ###

BEARは[firePHP](https://addons.mozilla.org/en-US/firefox/addon/6149)に対応しています。
firePHPには[fireBug](https://addons.mozilla.org/ja/firefox/addon/1843)が必要です。PHPのエラーやajax時のデバック出力関数がコンソールに出力されます。

# アップグレード #
現在のバージョンを確認します。
```
$ pear list-all -c bear
```

他PEARパッケージ含めて全部アップグレード
```
$ sudo pear upgrade-all
```
またはBEAR&Pandaのみアップグレード
```
$ sudo pear upgrade bear/BEAR
$ sudo pear upgrade bear/Panda
```

※Unknown remote channel: pear.firephp.orgというエラーが出ればpear channel-discover として新規チャネルの登録が必要です。

最新バージョンは[Channelサイト](http://pear.bear-project.net/)でも確認できます。

upgradeされたBEARのバージョンを確認します。
```
$ bear -v
```

## クリーンアップグレード ##

通常、普通のアップグレードで問題はありませんが何かオカシイというときに
```
$ sudo pear uninstall bear/BEAR
$ sudo pear clear-cache
$ sudo pear install -a bear/BEAR-beta
```


##  ##
# クリーンインストール #
root権限がなくグローバルなPEAR環境を使えない場合や、現在の環境に影響を与えたくない場合、またはプロジェクト毎にPEAR環境をセットアップしたい場合にユーザー環境でBEAR環境を構築することができます。

※またバージョン管理システムやファイルシンクでdeployするためのクリーン環境の構築にも利用できます。

## インストール ##
pearrcをつくりPEARからインストールをはじめます。


ホームディレクトリにpearrcを置く場合。
```
$ cd ~
$ pear  config-create `pwd` .pearrc
$ mkdir <プロジェクトPEAR環境>;
$ cd <プロジェクトPEAR環境>;
$ pear -c ~/.pearrc config-set auto_discover 1
$ pear -c ~/.pearrc config-set preferred_state alpha
$ pear -c ~/.pearrc channel-update pear.php.net
$ pear -c ~/.pearrc install PEAR
$ pear -c ~/.pearrc channel-discover pear.bear-project.net
（開発）
$ pear -c ~/.pearrc install -a bear/BEAR-beta
（運用）
$ pear -c ~/.pearrc install bear/BEAR-beta
```

### bin\_dirの確認 ###
```
$ pear -c ./.pearrc config-show | grep bin_dir
PEAR executables directory     bin_dir          <プロジェクトPEAR環境>/pear
```
### アプリケーションプロジェクト作成 ###
```
$ <プロジェクトPEAR環境>/pear/bear init-app -c ./.pearrc <プロジェクトPATH>
```


## 確認 ##

以下のコマンドでローカルPEAR環境の詳細を知ることができます。

```
$pear -c ./.pearrc config-show
```

### .pearcの設置場所 ###
ユーザーホーム(~)に設置すると`-c ./.pearrc`が省略できます。

### 手動でのvendors/PEARの更新 ###
v0.8は依存PEARライブラリが外部登録されているので、クリーンなPEAR環境からv.08をインストールしてそのPEARを使います。
```
pear -c ./.pearrc install bear/BEAR-0.8.99f
```
