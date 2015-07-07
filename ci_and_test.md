# 導入 #

疎結合でリクエスト状態を持たない「リソース指向設計」と、モデル（リソース）のCLI利用、ページリソース等のBEARのリソース指向とサービスオブジェクトの自由な入れ替えが可能なサービスロケータがユニットテストを支援します。

またテストやCI（継続開発)がスムーズに利用できるよに、プロジェクトに設定ファイルが最初から用意されているので、プロジェクトを作成した段階でantやphpunitが使えます

※v0.9.0以降の機能です。

# 詳細 #

## 準備 ##

### PHP ###
ユニットテストやCIに必要なライブラリをインストールします。

```
$ sudo pear channel-discover pear.pdepend.org
$ sudo pear channel-discover pear.phpmd.org
$ sudo pear channel-discover pear.phpunit.de
$ sudo pear channel-discover components.ez.no
$ sudo pear channel-discover pear.symfony-project.com

$ sudo pear install pdepend/PHP_Depend
$ sudo pear install phpmd/PHP_PMD
$ sudo pear install phpunit/phpcpd
$ sudo pear install phpunit/phploc
$ sudo pear install PHPDocumentor
$ sudo pear install PHP_CodeSniffer

$ sudo pear install -a phpunit/PHP_CodeBrowser
$ sudo pear install -a phpunit/PHPUnit
$ sudo pear install phpunit/DbUnit
$ sudo pear install phpunit/PHPUnit_Selenium
```

  * [PHPUnit のインストール](http://www.phpunit.de/manual/3.6/ja/installation.html)
### Jenkins(CIサーバー) ###
> [PHPもやらなきゃJenkins](http://www.bear-project.net/blog/2011/05/php%E3%82%82%E3%82%84%E3%82%89%E3%81%AA%E3%81%8D%E3%82%83jenkins/)

### プロファイラ ###
実行時間の分析や[コールグラフ](http://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%BC%E3%83%AB%E3%82%B0%E3%83%A9%E3%83%95)の出力を行うには、下記の設定が必要です。

#### xhprof によるプロファイリング機能を使うには ####
xhprofはFacebookの開発したWebで利用可能なプロファイラです。
PHPエクステンション「Xdebug」をインストールします。

  * [Xdebug](http://xdebug.org/)
  * [XHPROF](http://pecl.php.net/package/xhprof)
  * [Graphviz](http://www.graphviz.org/)

BEARアプリケーションの[App.php]で、[App::init]内に、以下のようにスクリプトを呼ぶことで、xhprof機能が有効化されます。
```
include 'BEAR/Dev/Profile/script/start.php'; 
```

#### XH GUI によるプロファイリング機能を使うには ####
xhprof のブランチでプロファイルの履歴が利用可能なXH GUIも利用可能です。

BEARアプリケーションの[App.php]で、[App::init]内に、以下のようにスクリプトを呼ぶことで、XH GUI機能が有効化されます。（前述のxhprof を有効にしていた場合は、start.phpにstartxh.phpに置換することで、有効化されるように変更されます。）

```
include 'BEAR/Dev/Profile/script/startxh.php';
```
XH GUIはmysqlを利用します。path/to/BEAR/data/xhprof/にあるREADMEを参照してdbを用意します。

```
SET NAMES utf8;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
--  Table structure for `details`
-- ----------------------------
DROP TABLE IF EXISTS `details`;
CREATE TABLE `details` (
  `id` char(17) NOT NULL,
  `url` varchar(255) DEFAULT NULL,
  `c_url` varchar(255) DEFAULT NULL,
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `server name` varchar(64) DEFAULT NULL,
  `perfdata` mediumblob,
  `type` tinyint(4) DEFAULT NULL,
  `cookie` blob,
  `post` blob,
  `get` blob,
  `pmu` int(11) DEFAULT NULL,
  `wt` int(11) DEFAULT NULL,
  `cpu` int(11) DEFAULT NULL,
  `server_id` char(3) NOT NULL DEFAULT 't11',
  `aggregateCalls_include` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `url` (`url`),
  KEY `c_url` (`c_url`),
  KEY `cpu` (`cpu`),
  KEY `wt` (`wt`),
  KEY `pmu` (`pmu`),
  KEY `timestamp` (`timestamp`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```
### Code Sniffer ###
PHPCSにはデフォルトでPEAR, Zend, Squiz等のコード規約ファイルが用意されてますがその３つを元に再編成したBEARのコード規約xmlが用意されています。

```
$ cp -r path/to/BEAR/data/phpcs/BEAR path/to/PEAR/CodeSniffer/Standards/
$ phpcs --config-set default_standard BEAR
```

規約のインストールを確認します。
```
$ php -i
The installed coding standards are BEAR, MySource, PEAR, PHPCS, Squiz and Zend
```



### コードアナライザ ###
Zend Code Analyzer
以前ZendStudioに付属していたものですが、現在入手難です。（最新バージョンには付属しません）利用すれば未使用変数の検知などコードの分析ができます。

phpcsが使用します。あるなら以下のコマンドでsetします。
```
phpcs --config-set zend_ca_path /path/to/ZendCodeAnalyzer
```


### BEAR App.php ###

HTTP越しのテストを行う場合にはテスト対象のサイトではコンポーネントをテストされる用のものに変更します。以下のようにApp.phpで$bearModeでテストモードを持つのが良いでしょう。


```
            case 100:
                // for HTTP access UNIT test
                $app['core']['debug'] = false;
                $app['BEAR_Log']['__class'] = 'BEAR_Log_Test';
                $app['BEAR_Resource_Request']['__class'] = 'BEAR_Resource_Request_Test';
                break;
```

※`BEAR_Log`サービスの実クラスが`BEAR_Log_Test`になり、テストクライアントから対象サイトのリソースリクエストやフォームバリデーション情報がHTTP経由で伝わるようになります。

またセッションが不要になる場合にはApp.phpまたはbootstrap.phpで以下のようにします。サービスレジストリの`BEAR_Session`サービスの実クラスを`BEAR_Session_Test`でセットすると際のセッションの書き込み読み出しは行われません。

```
BEAR::set('BEAR_Session', array('BEAR_Session_Test', array()));
```
## 設定ファイル ##

bear init-appで作成した新規プロジェクトファイルのトップ階層にant/JenkinsやPHPUnitで使うbuild.xmlとphpunit.xml.distが含まれてるのでプロジェクトファイルを作った時からant/やphpunitコマンドを引数なしで利用できます。
※従来のプロジェクトでもこのファイルを利用できます。その場合テストはtests/に置いてください。

新規example.appを作成して確認
```
$ cd /tmp
$ bear init-app example.app
$ cd axample.app
$ phpunit
$ ant
...
$ open build/api/index.html
$ open build/coverage/index.html 
$ open build/code-browser/index.html
$ open build/pdepend/*.svg
$ rm -rf example.app
```

## ユニットテスト ##

以下のテストが行えるクラスが用意されてます。

  * リソーステスト
  * ページHTMLテスト
  * ページValueテスト
  * ページのリソースリクエストテスト
  * フォームテスト

ページHTMLテストはページが出力するHTMLのテストで、ページバリューテストとはページにセットされた値のテストです。

このうちページバリューテストとフォームテストはHTTP越しの実際にGET/POSTをサブミットするテストになり、専用のサイトを用意する必要があります。

### リソーステスト ###
リソース単体でのテストです。この例では対象リソースのcodeとbodyを確認しています。
```
class Test_Ro_Test_User_Test extends PHPUnit_Framework_TestCase
{
    /**
     * for MDB2
     *
     * @var bool
     */
    protected $backupGlobals = FALSE;

    /**
     * @var BEAR_Resource
     */
    public $resource;

    public function setUp()
    {
        $config = array('path' => __DIR__ . 'App/Ro');
        $this->resource =  new BEAR_Resource($config);
        $this->uri = 'Test/User';
    }

    /**
     * read User?id=1
     */
    public function testReadId1()
    {
        $params = array(
            'uri' => $this->uri,
            'values' => array('id' => 1),
            'options' => array()
        );
        $ro = $this->resource->read($params)->getRo();
        $this->assertSame(200, $ro->getCode());
        $body = $ro->getBody();
        $expected = "サンデー";
        $this->assertSame($expected, $body['name']);
    }
```
### ページHTML(DOM)テスト ###
### ページバリューテスト ###

ページの出力するHTMLをテストします。htdocs/下にあるページをpage://のスキーマを持つページリソースとして扱い出力モードをHTMLにしてHTMLを検査します。

uriはページのクラスです。メソッドがreadのときでonInit($args)がコールされます。valuesで指定した値が$argsに渡されます。outputモードがhtmlの時はdsiplay()でレンダリングしたHTML、valueの時はページにsetされた内容が結果になります。

read以外のcreate, update, deleteメソッドの場合はonAction($submit)がコールされます。その場合valuesで指定した値が$submitに渡されます。メソッドの違いはありません。

HTMLのDOM要素取得は`Zend_Dom_Query`を利用した`BEAR_Test_Query`クラスを用います。CSSセレクタ（JQueryと同じ方式)でDOMを指定します。FireBugを用いれば簡単に表示されたページからCSSセレクタを取得することができます。

ページングもサポートしています。以下のサンプルでhtdocs/resource/link/pager.phpページの`Resource_Link_Pager`ページクラスで出力されるページのうち２ページ目をテストに現れる要素をテストしています。

```
class BEAR_resources_Test extends PHPUnit_Framework_TestCase
{
    public function setUp()
    {
        $this->_resource = BEAR::dependency('BEAR_Resource');
        $this->_query = new BEAR_Test_Query;
    }

    /**
     * リンクとテンプレート指定されたリソースにページャー付
     *
     */
    public function testResourceWithTemplateAndLinkAndPagerWithPage2()
    {
        $params = array(
            'uri' => 'page://self/resource/link/pager',
            'values' => array(),
            'options' => array(
                'output' => 'html',
                'page' => 2
            )
        );
        $html = $this->_resource->read($params)->getBody();
        $xml = $this->_query->getXml($html, 'html#beardemo body div.content ul li.blog');
        $expected = '<li class="blog">Athos Blog</li>';
        $this->assertSame($expected, $xml[0]);
        $xml = $this->_query->getXml($html, 'html#beardemo body div.content div#blog ul li.entry span.title');
        $expected = '<span class="title">Go</span>';
        $this->assertSame($expected, $xml[0]);
    }
```

### ページのリソースリクエストテスト ###

Cookieやクエリー等のHTTPリクエストの条件によってページがリソースにどういうリクエストを行うかというページのリソースリクエストのテストが行えます。MVCでいうとコントローラーのテストになるかと思います。

リソース結果や実装によらずに、特定HTTPリクエストによるページの働き（リソースリクエスト）をテストします。

下の例は`?id=1`のクエリーを付けてリクエストしたページが内部のROリソースに対して`read Entry?id=1`のリクエストを出しているかをテストしているコードです。`BEAR_Test_Client`はPEAR::HTTP\_Request2を継承しているクラスです。CookieやHeader等様々なリクエスト状態を作り出す事ができます。

```
class Page_Db_Select_Item extends PHPUnit_Framework_TestCase
{
    /**
     * @var BEAR_Test_Client
     */
    protected $client;

    public function setUp()
    {
        $this->client = new BEAR_Test_Client;
        $this->uri = 'http://bear.demo/db/select/item.php';
    }

    /**
     * ?id=1
     */
    public function testId1()
    {
        $log = $this->client->request('GET', $this->uri . '?id=1')->getResourceRequestLog();
        $expected = 1;
        $this->assertSame($expected, count($log));
        $this->assertSame('read Entry?id=1', $log[0]);
    }
}
```

### フォームサブミットテスト ###
フォームに実際に値をサブミットして内部のフォームのバリデーションをテストします。画面に表示されたエラー出力などを利用するのではなく、内部のフォームバリデーション結果情報をHTTP経由で取得します。

このためHTTPの制約を受けるのでより現実に即したサブミットテストが可能です。

以下のサンプルではフォームのバリデーションNGの確認をしています。フォームエラーの時に表示される文字列でエラーの種類の判断しています。
```
class Page_Form_Simple_Index_Test extends PHPUnit_Framework_TestCase
{
    /**
     * @var BEAR_Test_Client
     */
    public $client;

    public function setUp()
    {
        $this->client =  new BEAR_Test_Client;
        $this->uri = 'http://bear.demo/form/simple/index.php';
    }

    /**
     * OK
     */
    public function testPageSimpleIndexValidSubmit()
    {
        $submit = array('name' => '熊太郎', 'email' => 'kuma@bear-project.net');
        $errors = $this->client->request('POST', $this->uri, $submit)->getFormErrors();
        $this->assertSame(0, count($errors));
    }

    /**
     * No input
     */
    public function testPageSimpleIndexNoInputSubmit()
    {
        $submit = array('name' => '', 'email' => '');
        $errors = $this->client->request('POST', $this->uri, $submit)->getFormErrors();
        $this->assertSame(2, count($errors));
        $expected = '名前を入力してください';
        $this->assertSame($expected, $errors['name']);
        $expected = 'emailを入力してください';
        $this->assertSame($expected, $errors['email']);
    }

    /**
     * Invalid email
     */
    public function testPageSimpleIndexInvalidEmailFormatSubmit()
    {
        $submit = array('name' => '熊太郎', 'email' => 'kuma@@bear-project.net');
        $errors = $this->client->request('POST', $this->uri, $submit)->getFormErrors();
        $this->assertSame(1, count($errors));
        $expected = 'emailの形式で入力してください';
        $this->assertSame($expected, $errors['email']);
    }
}
```

複数のフォームがあるページではフォームの名前を指定します。

```
class Page_Form_Multi_Index_Test extends \PHPUnit_Framework_TestCase
{
    /**
     * @var BEAR_Test_Client
     */
    protected $client;

    public function setUp()
    {
        $this->client =  new BEAR_Test_Client;
        $this->uri = 'http://bear.demo/form/multi/index.php';
    }

    /**
     * Login OK
     */
    public function testPageSimpleIndexValidLoginSubmit()
    {
        $submit = array('id' => 'kuma', 'password' => 'kuma777');
        $isValidSubmit = $this->client->request('POST', $this->uri, $submit, 'login')->isValidSubmit();
        $this->assertTrue($isValidSubmit);
    }

    /**
     * Reminder OK
     */
    public function testPageSimpleIndexValidReminderSubmit()
    {
        $submit = array('email' => 'kuma@example.com');
        $isValidSubmit = $this->client->request('POST', $this->uri, $submit, 'reminder')->isValidSubmit();
        $this->assertTrue($isValidSubmit);
    }
}
```