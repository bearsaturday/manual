# Introduction #

## Key Concept ##

  * Intuitiveness 直感性
  * Drivability 操縦性
  * Separation of concern 関心の分離
  * Minimalism 禅

## Software Design ##

  * オブジェクトモデルを前提としない、ステートレスリクエストのリソースモデル。[サービスレイヤー](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?ServiceLayer)が内部APIとして機能。
  * ページコントローラ経由の必要がない独立したリソースモデル。
  * コンポーネント間の接続はHTTPがモデルのリソース指向アーキテクチャ。
  * イニシャライザ（インジェクター)で[Dependency Pull](http://www.google.co.jp/url?sa=t&source=web&cd=1&ved=0CB0QFjAA&url=http%3A%2F%2Flife.neophi.com%2Fdanielr%2Ffiles%2FInversionOfControl.pdf&rct=j&q=Dependency%20Pull&ei=ldEJTrCLIqGDmQW9762TAQ&usg=AFQjCNH03mTNlZXMA-QgdEP4izqzJuB88g&sig2=wjawUH6RbA3pKA9M2pu6Rg&cad=rja)を用い外部からの切り替えとサービスロケータ利用で依存オブジェクトを制御。
  * ページコントントローラー。ページ間が共有するものを最小化。
  * リソースモデルのアクセスにプロキシを使う[アスペクト指向プログラミング](http://ja.wikipedia.org/wiki/%E3%82%A2%E3%82%B9%E3%83%9A%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0)。
  * 問合せから副作用の心配を取り除く[Side-Effect-Free Functions パターン : 副作用をなくす設計](http://masuda220.jugem.jp/?eid=335)。
  * [PEAR-Style Pseudo-namespace(PSR-0 Compatible)](http://groups.google.com/group/php-standards/web/psr-0-final-proposal?pli=1)
  * コンポーネントのあり方よりむしろ接続に注目しHTTPをモデルとする。
  * [Push-based & Pull-based](http://en.wikipedia.org/wiki/Web_application_framework#Push-based_vs._Pull-based)
  * リソースはコントローラでもありモデルでもある[レイヤードアーキテクチャ](http://en.wikipedia.org/wiki/Multitier_architecture)。
  * ビューに対するレイジーセット。
  * 継承関係をもつUAに対するViewの[OCP](http://ja.wikipedia.org/wiki/%E9%96%8B%E6%94%BE/%E9%96%89%E9%8E%96%E5%8E%9F%E5%89%87)


## プログラミングパラダイム ##

  * オブジェクト指向プログラミング (OOP)
  * 属性指向（注釈指向）プログラミング (AOP)
  * アスペクト指向プログラミング (AOP)
  * [リソース指向アーキテクチャ](http://en.wikipedia.org/wiki/Resource-oriented_architecture)(ROA)

### [PoEAA](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?CatalogOfPofEAA) ###

  * [ページコントローラ](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?PageController)
  * [テンプレートビュー](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?TemplateView)
  * [2(3..n)ステップビュー](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?TwoStepView)
  * [レジストリ](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?TemplateView)
  * [依存性の注入とサービスロケーター](http://kakutani.com/trans/fowler/injection.html)
  * [データトランスファーオブジェクト](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?ValueObject)
  * [レイジーロード](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?LazyLoad)
  * [クエリーオブジェクト](http://capsctrl.que.jp/kdmsnr/wiki/PofEAA/?QueryObject)

### 他技術 ###
  * [ユニバーサルコンストラクタ](http://paul-m-jones.com/archives/1500)
  * [ステートレスインターフェイス](http://hamasyou.com/archives/000345)

## インスパイアされたソフトウエア ##
  * PEAR
  * [SolarPHP](http://solarphp.com/)
  * Zend Framework
  * Click Framework (Java)
  * Guice (Java)
  * jQuery

## 書籍 ##
  * [Clean Code アジャイルソフトウェア達人の技 by Robert C. Martin](http://t.co/2bIT9tW)
  * [Webを支える技術 -HTTP、URI、HTML、そしてREST (WEB+DB PRESS plus) by 山本 陽平](http://t.co/S6DnJyJ)
  * [エンタープライズ アプリケーションアーキテクチャパターン (Object Oriented Selection) by マーチン・ファウラー](http://t.co/6asovEj)
  * [エリック・エヴァンスのドメイン駆動設計 (IT Architects’Archive ソフトウェア開発の実践) by エリック・エヴァンス](http://t.co/FqC2GV1)
  * [ThoughtWorksアンソロジー ―アジャイルとオブジェクト指向によるソフトウェアイノベーション by ThoughtWorks Inc.](http://t.co/QtteJKR)
  * [オブジェクト指向における再利用のためのデザインパターン by エリック ガンマ](http://t.co/r2SvHoF)
  * [Head Firstデザインパターン ―頭とからだで覚えるデザインパターンの基本 by Eric Freeman](http://t.co/lUop6my)
  * [Webアプリケーション設計・実装のためのフレームワーク活用の技術 by 古川 正寿 ](http://t.co/vANcMNB)
  * [エンジニアのためのJavadoc再入門講座 現場で使えるAPI仕様書の作り方 by 佐藤 竜一 ](http://t.co/TLTUvvR)
  * [オブジェクトデザイン (Object Oriented SELECTION) by レベッカ・ワーフスブラック](http://t.co/XDbUIk4)
  * [Code Complete第2版〈上〉―完全なプログラミングを目指して by スティーブ マコネル](http://t.co/4FBtxuR)
  * [Code Complete第2版〈下〉―完全なプログラミングを目指して by スティーブ マコネル](http://t.co/Xv9xa5y)
  * [ビューティフルコード by Brian Kernighan](http://t.co/pjt0kIN)
  * [ビューティフルアーキテクチャ by Diomidis Spinellis ](http://t.co/CV34x9r)
  * [コンピュータプログラミングの概念・技法・モデル(IT Architect' Archiveクラシックモダン・コンピューティング6) (IT Architects’Archive ... by セイフ・ハリディ](http://t.co/ya3ykvQ)
  * [インターフェイス指向設計 ―アジャイル手法によるオブジェクト指向設計の実践 by Ken Pugh](http://t.co/uPkVWFU)
  * [Real-World Solutions for Developing High-Quality PHP Frameworks and Applications](http://t.co/Y032BU3)

## ブログ記事 ##

### ソフトウエアアーキティクチャパターン ###
  * [GUI-MVCとWeb-MVCの違い](http://d.hatena.ne.jp/yojik/20091019/1255963600)
  * [matarillo.com UIパターン](http://matarillo.com/general/uipatterns.php)
  * [What is MVC? – O’Reilly ONLamp Blog](http://www.oreillynet.com/onlamp/blog/2007/06/what_is_mvc.html)
  * [Seasar アプリケーションアーキテクチャ](http://sastruts.seasar.org/featureReference.html#Architecture)
  * [MVC vs. MVP – darron schall](http://www.darronschall.com/weblog/2004/06/mvc-vs-mvp.cfm)
  * [技術講座 ](.md) Domain-Driven Designのエッセンス
  * [ドメインの関心事　それ以外の関心事 | システム設計日記](http://masuda220.jugem.jp/?eid=293)
  * [Kore Nordmann Why active record sucks](http://kore-nordmann.de/blog/why_active_record_sucks.html)
  * [ActiveRecord does not suck](http://karwin.blogspot.com/2008/05/activerecord-does-not-suck.html)
  * [Presentation-abstraction-control, Hierarchical-Model-View-Controller](http://en.wikipedia.org/wiki/Presentation-abstraction-control)
  * [ひがやすを blog 3つのモデル - どのモデルを中心にするのか](http://d.hatena.ne.jp/higayasuo/20050913)
  * [WPF のための MODEL-VIEW-VIEWMODEL (MVVM) デザイン パターン](http://msdn.microsoft.com/ja-jp/magazine/dd419663.aspx)
  * [Model-view-presenter](http://en.wikipedia.org/wiki/Model-view-presenter)
  * [Digital Romanticism Greg Young流CQRS - Mark NijhofAdd Star](http://d.hatena.ne.jp/digitalsoul/20100712/1278886009) - [原文](http://elegantcode.com/2009/11/11/cqrs-la-greg-young/)
### REST ###
  * [Architectural Styles and the Design of Network-based Software Architectures, Chapter 5](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
  * [yohei-y:weblog  REST入門](http://yohei-y.blogspot.com/2005/04/rest_23.html)

### DI/AOP ###
  * [Inversion of Control コンテナと Dependency Injection パターン](http://kakutani.com/trans/fowler/injection.html)
  * [Learning About Dependency Injection and PHP](http://ralphschindler.com/2011/05/18/learning-about-dependency-injection-and-php)
  * [google-guice > User's Guide > Motivation](http://code.google.com/docreader/#p=google-guice&s=google-guice&t=Motivation)
  * [http://www.infoq.com/jp/news/2009/08/dependency-injection-javaee6](http://www.infoq.com/jp/news/2009/08/dependency-injection-javaee6)
  * [Guice による依存性注入](http://www.ibm.com/developerworks/jp/java/library/j-guice.html)
  * [Dependency Injection the JSR 330 way](http://matthiaswessendorf.wordpress.com/2010/01/19/dependency-injection-the-jsr-330-way/)
  * [Guice（ジュース）を早飲みしすぎていませんか？](http://www.infoq.com/jp/articles/drinking-your-guice-too-quickly)
  * [Zend\_Application(2) /Zend FrameworkにおけるDIコンテナ活用のメリットについて](http://d.hatena.ne.jp/noopable/20090412/1239484575)
  * [noopな日々モデルもしくはサービスレイヤーに関する補足](http://d.hatena.ne.jp/noopable/20100309/1268103347)
  * [noopな日々 DIコンテナとフレームワークの関係のあるべき姿とは？](http://d.hatena.ne.jp/asip/20070518#1270775397)

### PHP ###
  * [PHPundamentals Series: A Background on Statics](http://ralphschindler.com/category/articles/phpundamentals)
  * [Paul M. Jones Benchmarking Slides from PHPBenelux 2011](http://paul-m-jones.com/archives/1727)
  * [Running The Symfony 2 Benchmarks](http://paul-m-jones.com/archives/1222)

### Design ###

  * [New Considered Harmful](http://c2.com/cgi/wiki?NewConsideredHarmful)


BEARフレームワークの設計・実装でインスパイアされたり参考にしたりしたものを記しています。