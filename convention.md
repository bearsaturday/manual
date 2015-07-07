# 導入 #

BEARのコード規約はPEARとZendの規約を組み合わせたものです。PHPDocのコメント部分の書式はPEAR、コメント以外のコードはZendの規約とそれぞれ厳しい方を採用しています。コードのチェックには[PHP\_CodeSniffer](http://pear.php.net/manual/ja/package.php.php-codesniffer.php)を使用しPEAR/Zend双方の規約でチェックしています。

# 詳細 #
## ファイル配置とクラス名 ##
PEARやZendのライブラリのようなファイルパス、クラス名にしています。

### 単一ファイルファイルパッケージ ###
例)`<パッケージ名>.php`
  * Error.php
### 複数のクラスで構成されるパッケージ ###
例）補助クラス使用
  * Emoji.php
  * Emoji/Map.php (`Emoji_Map`クラス)

例）インターフェイス使用
  * Ro.php
  * Ro/Interface.php(`Ro_Interface`インターフェイス)

例）Adpterパターン使用
  * Cache.php
  * Cache/Adapter.php （`BEAR_Cache_Adapter`アブストラクトクラス）
  * Cache/Adapter/Memcache.php（`BEAR_Cache_Adapter_Memcache`コンクリートクラス）


## Zend規約 ##

詳細は[Zend Framework PHP 標準コーディング規約](http://framework.zend.com/manual/ja/coding-standard.html)で。以下はPEARになくZend規約だけにあるものの抜粋です。

  * 変数名、プロパティ名にアンダースコア(` _` )は含めない。数字非推奨。×$md5 ○$mdFive
  * 定数宣言の場合はdefineでなくconstを使う。
  * 1行120文字(80行以上でWARNING, 120でERROR)
  * PHPのみのコードは最後の ?> を含めない
  * インターフェースは名前の最後に`_Interface`を付与する。

## PEAR規約 ##

詳細は[PEAR規約](http://pear.php.net/manual/ja/standards.php)参照。以下はPEAR規約のエラーチェックをパスするPHPDocのコメント例。
```
<?php

/**
 * BEAR
 * 
 * PHP versions 5
 *
 * @category  BEAR
 * @package   BEAR_Cache
 * @author    Akihito Koriyama <koriyama@users.sourceforge.jp>
 * @copyright 2008 Akihito Koriyama  All rights reserved.
 * @license   http://opensource.org/licenses/bsd-license.php BSD
 * @version   SVN: Release: $Id:$
 * @link      http://api.bear-project.net/BEAR_Cache/BEAR_Cache.html
 */

/**
 * Memcacheキャッシュクラス
 * 
 * BEAR_Cache_Adapter抽象クラスをPECL::Memcacheをエンジンに実装しています。
 * 
 * @category  BEAR
 * @package   BEAR_Cache
 * @author    Akihito Koriyama <koriyama@users.sourceforge.jp>
 * @copyright 2008 Akihito Koriyama  All rights reserved.
 * @license   http://opensource.org/licenses/bsd-license.php BSD
 * @version   SVN: Release: $Id:$
 * @link      http://api.bear-project.net/BEAR_Cache/BEAR_Cache.html
 */
class BEAR_Cache_Adapter_Memcache extends BEAR_Cache_Adapter
{

    /**
     * キャッシュを保存
     *
     * 指定したキーでキャッシュを保存します。
     *
     * @param string $key   キャッシュキー
     * @param mixed  $value 値
     * 
     * @return bool
     */
    public function set($key, $value)
    {
```
  * ファイルコメントブロックとクラスコメントブロックは別々に必要
  * 間隔は最大の長さの単語＋1スペースで縦を揃える。
  * @paramの前後は1行空行が必要。
  * 入力がないときは@paramは省略できるが、@returnは必須。
  * 連想配列が入力でキー別に説明が必要ならコメント欄に記入。

ここまでするとphpcsのPEAR規約で検証をかけたときにエラーが出ません。

## PEAR規約とZend規約のコンフクリクト ##

Zend規約はPEAR規約をより厳密にしたもののようですが一部競合します。

  * 相違のあるものについてはZend規約を優先させています。(PHPのみの場合?>で閉じないなど）
  * protectedの変数名についてもアンダースコアを付けています。ここはPEAR規約と大きく違うところです。
```
    /**
    * イメージアトリビュート
    * 
    * @var string
    */
    protected $_srcAttr;
```

## PEAR/Zend両規約対応 ##

以下のようなインデント、スペースのコードで両規約に対応します。
```
// if文
if ($nameOne == 'Obama') {
    $x = 1;
}
// switch文
switch ($align) {
case 'center' :
    $x = 1;
default:
}
// foreach文
foreach ($array as $row) {
    $data[] = $row;
} 
// for文
for ($i = 0; $i < 10; $i++) {
    $data[$i] = array(1, 2, $i);
}
```
  * 括弧の前後のスペースを1つ開けるのはZend規約
  * switchとcaseを揃えるのはPEAR規約

## フォーマット ##
BEAR/data/docs/eclipse/BEAR Convention.xmlで定義されたZendStudioのフォーマッティング機能で全コードを統一フォーマットしています。このxmlファイルを使ってフォーマットするとPEAR規約、Zend規約双方でスペーシングやブランケットなどの規約エラーは出なくなります。（これから@要素を除くと規約エラーになります）

## Solar規約 ##
名前の付け方、ファイル配置などは[Solar規約](http://solarphp.org/manual:project_standards:naming_conventions)も参考にしています。

## Vim設定例 ##

Vimでコードを書く際の基本的な設定例です。
```
set nocompatible
syntax on
filetype indent on
set autoindent smartindent
set nocindent
set smarttab
set expandtab
set shiftround
autocmd FileType php setlocal tabstop=4 shiftwidth=4 softtabstop=4
set backspace=indent,eol,start
set encoding=utf-8
set fileencodings=utf-8
set listchars=tab:>.,trail:_,extends:>,precedes:<
```

1行目はvi互換モードをOFFにしています。

2行目でファイルの構文ハイライトをONにしています。

3〜9行目でインデントの設定を行っています。

10行目でbackspaceでインデントや改行の削除ができるよう設定をしています。

11,12行目でエンコーディングの設定をしています。

13行目でタブや空白を記号で表示するよう設定しています。

### おすすめのプラグイン ###
  * syntastic(構文チェック) https://github.com/scrooloose/syntastic
  * neocomplete(補完、vim7.4以上) https://github.com/Shougo/neocomplete.vim
  * neocomplcache(補完、vim7.3) https://github.com/Shougo/neocomplcache.vim