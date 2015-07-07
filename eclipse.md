# 導入 #

Eclipse 2.6(Helios) + PDT 2.2をインストールする例です

# 詳細 #

## eclipse本体 ##
http://www.eclipse.org/pdt/downloads/ (PDT-All-in One) または http://eclipse.org/downloads/ からPDTをダウンロード、解凍してアプリケーションフォルダに移動します。

## 日本語化 ##

日本語化には1) **Pleiades** による方法と2) blanco Frameworkが提供している 2) **Eclipse 日本語化言語パック (サードパーティ版)** による方法があります。PleiadesはAOPによる日本語化でプラグインが多言語に対応してないものも含めて日本語化できるメリットがある一方、AOPによる翻訳のためCPUコストの負担が考えられます。2)の日本語化言語パックはEclipseの標準的な他言語対応の方式で、対応が他言語化が考えられているプラグインに限られる一方、英語版と変わらない軽さです。またeclipse.iniの設定も必要ありません。

ここでは2)の方法でインストールします。

  1. http://sourceforge.jp/projects/blancofw/wiki/nlpack.eclipse からEclipse にあるバージョンのnlpack（言語パック)をダウンロード （2011.11.30現時点では nlpack.eclipse.helios-M4-I200912232200）
  1. 解凍してできたecliseフォルダを「インストールした本体のdropinフォルダ」にいれてecliseを起動すると日本語化されます。

## PDT ##

環境設定 > インストール/更新 > 使用可能なソフトウエア > 追加 で
http://download.eclipse.org/tools/pdt/updates/2.2/milestones/
を名前「PDT 2.2」等にして追加します。

"新規ソフトウエアのインストール"で上記で追加したサイトを選びPDT2.2を更新します。

## SVN ##
ソフトウエア更新からSubversiveを選んでインストール。立ち上げたらSVN ConnnectorをSVN Kit 1.3（もしくは使用するsvnにあったバージョン）を選びます。

## 設定 ##

### 一般 ###

  * 一般 > エディター > テキスト・エディター > タブでスペースを挿入
  * 一般 > ワークスペース > テキスト・ファイル・エンコード > UTF-8
  * 一般 > ワークスペース > 新規テキストファイルの区切り文字 > LF
  * PHP > Code Style > Formatter > Spaces 4
  * PHP > Editor > Typing > Close PHP tagをオフに
  * Web > Css Files > Encoding > UTF-8
  * Web > HTML Files > Encoding > UTF-8

### デバッガー ###

  * PHP > Debug > PHP Debugger > Xdebug

## MakeGood インストール ##

### Stagehand\_TestRunner をインストール ###
```
pear channel-discover pear.piece-framework.com
pear install piece/stagehand_testrunner
```
詳細は以下を参照
  * [Stagehand\_TestRunner - テスト駆     動開発のためのテストランナー - Piece Framework](http://redmine.piece-framework.com/wiki/stagehand-testrunner/Ja_Overview)
  * [PHP で快適なテスト駆動開発を - Stagehand\_TestRunner の特徴と使い方を知る | ITEMAN Blog - アイテマンブログ](http://iteman.jp/blog/2009/10/php---stagehand-testrunner.html)

### makegood ###
更新サイトを追加してプラグインとしてインストール。詳しくは以下を参照。
  * http://redmine.piece-framework.com/projects/makegood/wiki/Ja_UserGuide