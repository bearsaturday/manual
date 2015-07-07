# 導入 #

ページからテンプレートにHello Worldという文字列をアサインして表示する例です。


Note:
> リソースは使っていません。

# ファイル #
使用するファイルは2つです。一つはhtdocs/以下にある「ページファイル」で、実際のページの処理をページクラスとして実装します。もう一つは「ビューファイル」、Smartyテンプレートファイルです。

BEARはページ指向のフレームワークです。htdocs/にあるページクラスファイルが直接実行されます。イベント駆動でページクラスの処理が実行されページを表示します。

このhello.phpでは初期化onInit()のところで、「Hello Wolrd」という文字列をテンプレートにアサインしています。その後上位クラスがもつonOutput()メソッドがonInit()の後にコールされ、クラス名から自動的に特定したテンプレートに変数がアサインされページが表示しています。

ページファイル
htdocs/hello.php
```
<?php 

 require_once('App.php');

 /**
  * HelloWorldページ
  * 
  */
 class Page_Hello extends App_Page
 {    
     public function onInit(array $args)
     {
         $this->set('message', 'Hello World');
     }

     public function onOutput()
     {
         $this->display();
     }
 }

App_Main::run('Page_Hello');
?>
```

テンプレートファイル
App/views/pages/hello.tpl
```
 <html>
 <head>
   <title>{$message}</title>
 </head>
 <body>
   {$message}
 </body>
 </html>
```

## ページファイル ##

システム初期化、ページクラス定義、実行の3つ記述がしてあります。これはどのページファイルでも同じです。原則的にページが違うとページクラス定義が変わるだけです。

ページクラスはイベントで呼ばれる処理を記述します。接頭辞がonのメソッドがイベントでコールされます。

２つのPHP Doc型のコメントはPHP Documentor用のコメントです。この形式で書いておきPHP Documentorを使えばページクラスAPIのドキュメントが簡単にできます。ページ単位で処理の内容を把握するのに役立ちます。ファイル全体で1 つ、ページクラスファイルで1つと２つ必要です。ドキュメント不要の場合には省略できます。

```
require_once('App.php');
```

App.php内でフレームワークの初期化を行います。全てのページファイルに記述します。

```
class Page_Hello extends App_Page
```
ページクラス定義です。App\_Pageはアプリケーション共通ページクラスです。

```
public function onInit(array $config){
...
}
```

ページクラスの初期化です。一般的な表示のページではonInit()でリソースアクセス(read)を行い、ページにそのリソースをsetします。表示以外のすべての処理（たとえばフォーム）はここに記述します。

```
function onOutput()
{
    $this->display(); // このページでは$this->display('hello/world.tpl');と同じ
}
```

このhello.phpでは最上位のBEAR\_Pageクラスで定義されている上記のonOutputハンドラが省略されています。 display()はテンプレートのファイルパスを引数としてもちますが、省略した場合はページクラス名から生成されます。クラス名のアンダースコアを /（スラッシュ）にしたものがテンプレートファイルパスになります。
```
App_Main::run('Page_Hello');
```
ページクラス名を指定してBEAR\_Main::run()で実行します。

### まとめ ###

ページファイルは以下の３つで構成されてます。

  1. App.phpでBEAR初期化
  1. ページクラスを定義
  1. 定義したページクラスをrun