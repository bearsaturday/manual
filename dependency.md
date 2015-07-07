# 導入 #

BEARではDependency Injection (依存性の注入)というプログラミングパラダイムを全般的に採用しています。DIはコンポーネント間の関係性を疎にし、利用する側での利用するサービスの実装への依存を取り除こうとする技術です。

BEARのDIはPaul M Jones氏のSolarPHPというフレームワークのDI技術を基礎に拡張したもので、一般にJavaのフレームワークで使われているDI実装やPHP 5.3+用に登場してきたPHPフレームワークのDIと比べるととても簡易的で学習や実装のコストも低いものです。
（PHP 5.2時代のフレームワークの多くはDIの仕組みを持たず、あるいは持っていても利用は限られている現状がありました）

ですがこの技術のおかげでフレームワークコンポーネントのかなりの部分がフレームワークのソースにタッチすることなしに簡単にユーザーのものと差し替えられるようになっています。テストやデバック、開発もDIによってより効率化され導入効果は採用コストを大きく上回るものと考えています。

[スライド](http://www.slideshare.net/akihito.koriyama/bear-di)


## はじめに ##
_コード再利用を推進するために最も良いことは、インターフェースを実装から分離することです。Eric Raymondは『Art of UNIX Programming』の中で、UNIX®哲学の幾つかを次のように指摘しています。_

  * モジュラー化の原則: 単純な部分を書き、きれいなインターフェースで接続する
  * 分離の原則: ポリシーを機構から分離し、インターフェースをエンジンから分離する
  * 表現の原則: ナレッジをデータの中に入れ込むことによって、プログラム・ロジックは愚かで堅牢となるようにする

_そして分離のための技術の最新版、DI（dependency injection: 依存性注入）は、上記の理想を反映したものです。_

[J2EEアプリケーションでの分離を新しい角度から見る by Neal Ford より引用](http://www.ibm.com/developerworks/jp/opensource/library/os-ag-ioc1/)

BEARのDIも分離の原則のために使われています。

## 利用する側の依存を取り除く ##

_「DIのような話題を理解することが難しいのは、DIの話題とは無関係な他の話題とDIとが、非常に強く結びついているためです。」_

依存を分離集中させ、結合を疎にする技術を説明するのに皮肉な話です。では「利用する側の依存」とはなんでしょうか？具体例をあげながら説明したいと思います。

以下はPDOでDBインスタンスを生成するコードです。

```
  $pdo = new PDO("mysql:host=localhost; dbname=pdotest","root", "password");
```

利用するコード（クライアント）はこのクラスの名前を直接指定してパラメータを順に記述しnew演算子でインスタンス生成します。クライアントはPDOの実装に依存していて、PDOの初期化パラメータが変わったり、新しい"PDO2"を使おうとしたときにクライアントのコードは変更の必要があります。クライアントはサービスに依存しています。

ここで「分離の原則」のDIの登場です。依存性の注入、つまり利用側の依存を外から注入します。直接PDOのインスタンスを作る代わりに、DBインターフェイスを実装したPDOクラスを生成して外部の専用ツールに利用可能にしてもらいます。PDO2の開発ではDB
インターフェイスを実装し、クライアントの利用方法に変わりがないことを保証します。クライアントはPDOの実装への依存を取り除くことができます。実装の変わりにDBインターフェイスに依存します。

どのクライアントにどのサービスを利用可能にするか（注入するか）の指定はフレームワークによって様々です。XMLで記述するもの、コードで記述するもの、メソッドのアノテーションや命名規則でやるものなど様々です。主なDIコンテナFWではXMLを使うことが多いようですが、GoogleのGuice（YouTube, Ad Sense等)はコードベースです。

# BEARのDI #

## インスタンスの生成 ##

### BEAR::factory() ###

通常PHPでインスタンスの生成は以下のようにnew演算子で行います。

```
$myObj = new HogeClass($config);
```

あるいはクラス自身が自身の生成をするメソッドを利用します。(Factoryパターン)
```
$myObj = HogeClass::getInsetance($config);
```

BEARではBEAR::factoryメソッドを使います。

```
$myObj = BEAR::factory('HogeClass', $config);
```

newはその時に与えられた引数で、直接記述されたたクラス名のクラスのインスタンス化を行います。

対してBEAR::factory()は予めBEAR::init()で設定した「クラスをどのように生成するかを記述した設定値」に基づいてインスタンスが作成されます。利用するときはnewの時と同じようにクラス名と引数を指定するだけです。

ただし引数は連想配列ひとつだけです。

```
$obj = BEAR::factory($class, $config);
```

以下が生成動作です。

  1. `__class`パラメータでクラス名が指定されていればそのクラス名、なければ第一引数の$classがクラス名になります。
  1. その時に与えられた引数とアプリケーションが持つデフォルトの値を合成しコンストラクタのパラメータになります。
  1. `__injector`と指定された名前、省略されてる場合は`onInject`と名前のメソッドがあるかを調べて、ある場合はコンストラクタの次に初期化メソッドとしてコールされ、通常そのクラスで利用するオブジェクトのインスタンス化とプロパティへの代入が行います。

以上です。

例)

```
$obj = BEAR::factory('AppDb', array('host'=>'testhost'));
```

このコードは生成の設定に指定がないときは

```
$obj = new AppDb(array('host'=>'testhost'));
```

と同じです。

```
# 設定 app.yml
AppDb:
  __class: MyDb
  __injector: onTestInject
  id: user
  password: pass
  host: localhost
```

以上の設定ファイルで以下のコードが利用されるとき
```
$obj = BEAR::factory('AppDb', array('host'=>'testhost');
```

以下と同じコードで生成されたインスタンスが得られます

```
$obj = new MdDb(array('id'=>'user', 'password'=>'pass', 'host'=>'testhost');
$obj->onTestInject();
```

実際に利用されるクラスが代わり、引数が一部だけ変わっても他の引数は引き続き利用され、利用コードに

  1. 実装クラス名の決定
  1. 生成メソッドの決定（コンストラクタかfactory()メソッドか）
  1. コンストラクタパラメータの決定
  1. 初期化メソッドの決定と実行

が行われます。特徴は上記の設定は利用コードには全く現れないということです。
以下の利用コードは同じような単純さを持ちますが、BEAR::factory()はnewと違って外部からどのようにインスタンスが生成されるかをある程度決定することができます。

```
// newを使ったインスタンス生成
$db = new App_Db($config);

// BEAR::factory()を使ったインスタンス生成
$db = BEAR::factory('App_Db', $config);
```

### 初期化メソッド（インジェクターメソッド） ###

  * 初期化メソッド内で注入コード（サービスオブジェクトを取得してプロパティに代入）を記述します。このようにそのクラス内に利用するオブジェクトの代入のプロパティへの代入コードが記述してあるのが内部インジェクター、対して外部からインジェクトするものを外部インジェクターと呼びます。
  * 内部インジェクターは同じクラス内にあるため見通しが良く継承も使えますが、複数のクラスを横断するときは外部インジェクターを使う必要があります。
  * 外部インジェクターでは該当オブジェクトが渡され、そのプロパティに代入することになります。public以外のプロパティは[ReflectionMethod::setAccessible クラスリフレクション](http://www.php.net/manual/ja/reflectionmethod.setaccessible.php)を使ってアクセス権を変更する必要があります。


### BEAR::dependency() ###

BEAR::depedency()はインスタンス生成にBEAR::factory()を利用して、そのインスタンスの管理もしようとするものです。インスタンスの管理に後述する"サービスロケータ”を使います。

## サービスロケータ ##

サービス（オブジェクト）はレジストリと言われる場所に集中して保管され、サービス名を使って出し入れされます。レジストリにはオブジェクトしか保管できず、また保管されたオブジェクトは変更を加えることができません。

シングルトンパターンでは各クラスに保管されるインスタンスが、集中して管理される場所と考えればいいでしょう。各クラスがgetInstance()メソッドをもちシングルトンなどのインスタンス管理を行うかわりに、BEARのレジストリによるサービスロケータがインスタンスの管理を行います。

### サービスの利用 ###

'db'という名前で登録してるサービスを取り出します。
```
$db = BEAR::get('App_DB');
```

## サービスの登録 ##
### イーガー（即値)で登録 ###
インスタンス化はセットされるタイミングで行われ、そのサービスを利用するかどうかに関わらず生成コストは（当然）発生します。
```
$config = array('host' =>'localhost', 'port' => 3306);
$db = BEAR::factory('App_DB', $config);
BEAR::set('App_DB', $db);
```

### レイジー（遅延）で登録 ###

```
$db = array('App_Db', array('host' =>'localhost', 'port' => 3306));
BEAR::set('App_DB', $db);
```

レジストリにはインスタンスではなくインスタンス生成の方法が登録されます。レジストリから取り出されるBEAR::get()の時にインスタンス化が行われます。

### 登録と利用を一度に ###
```
$config = array('host' =>'localhost', 'port' => 3306);
$db = BEAR::dependency('App_Db', $config); //生成、登録して利用
// 2回目は生成は行われない。（シングルトン）
$db = BEAR::dependency('App_Db', $config); // 登録されたものから取り出して利用

```


## まとめ ##

```
$myObj = BEAR::dependency('My_Class', $config, $option);
```

上記のコードで以下の事が行われます。

  1. インスタンスの生成
  1. 初期化メソッドの実行による依存の注入（利用オブジェクトのプロパティへの代入）
  1. インスタンス管理（シングルトン or ）

$optionはどのようにインスタンスを生成管理するかというオプションです。シングルトンか毎回生成、利用オブジェクトのキャッシュや、インジェクターの指定のオプションがあります。

## 生成オプション ##

  * プロトタイプ（インスタンスを毎回生成）で生成するには$optionをfalseにするか$optionをarray('is\_sngleton'=>true)にすることでもできますが、BEAR::factory()の使用を推奨します。
  * 利用オブジェクトの永続化ができます。`$option`を`array('psersistent'=>true);`にすることで実現できます。スクリプトの初期化が終わった状態でオブジェクトはキャッシュされ、生成と初期化コストを節約することができます。

## コンストラクタ引数 ##

BEAR::Factory()の項で説明したようにコンストラクタは連想配列１つだけです。

  * コンストラクタで`_config`プロパティに格納します。どのクラスでも`$this->_config`でオブジェクト生成時の設定（コンストラクたの引数）を参照することができます。
**つまりデバック時などはどのクラスでも以下のコードでコンストラクタ引数が確認できます。**

```
p($this->_config);
```


# サンプルコード #

具体例で説明します。まずファイルの保存と読み込みができるファイルクラスを考えます。
はじめにファイルの読み書きのインターフェイスを用意します。

```
interface File_Access_Interface
{
    public function load($path);
    public function save($path, $body);
}
```

その実装クラスを用意します
```
File_Access implements File_Access_Interface
{
...
}
```

それを利用する`My_File`クラスがあるとします。このようにかきます。

```
class My_File extends BEAR_Base
{
    pritave $_file;

    public function onInject()
    {
        $this->_file = BEAR::dependency('File_Access', $config);
    }

    public function echoFile($path)
    {
        echo $this->_file->load($path);
    }
}
```

利用コードはこのようになります。

```
$myFile = BEAR::dependency('My_File');
$myFile->echoFile('hello.txt');
```

次に`File_Access`より高機能な`File_Remote_Access`クラスを開発するとしましょう。`File_Remote_Access`クラスは同じインターフェイスを持ちリモートファイルの読み書きもできるクラスすで。

```
    public function onInject()
    {
        $this->_file = BEAR::dependency('File_Remote_Access', $config);
    }
```

インジェクトコード（onInject）で注入しているサービスオブジェクトを変えても利用にメソッドのechoFileに変更はありません。利用が実装に依存せずにインターフェイスに依存しています。

インジェクターを複数もち生成時にインジェクターを選択することもできます。

```
class My_File extends BEAR_Base
{
    public function onInject()
    {
        $this->_file = BEAR::dependency('My_File', $config);
    }

    public function onInjectRemote()
    {
        $this->_file = BEAR::dependency('My_Remote_File', $config);
    }
}
$myFile = BEAR::dependecy('My_File', $config, array('injector'=>'onInjectRemote');
```

同様にモック（ダミーデータ、ダミークラス）のインジェクトにも使えます。
例えば外部サイトのサービスから値を利用するクライアントでは、まずモックで期待される値をインジェクトしてやりクライアントを開発、その後インジェクターを開発しモックから置き換えます。モックはそのまま残しておいて、トラブルがあったときにインジェクトに問題があるのか、クライアントに問題があるのかの切り分けが簡単にできます。「分離の原則」です。

クライアントコードの最初にサービスを用意するのが基本原則なのですが、それだとちょっとまずい場合があります - 用意したサービスを使わない場合です。その生成コストが無駄になります。

この問題にはオブジェクトの遅延ロードで対処します。他のクラスのオブジェクトを即時注入する方法と、他のオブジェクトを遅延ロードさせる方法と合わせて紹介します。いずれの場合もクライアントのコードに変化はなく利用の依存性が外から注入されています。


```
class My_File extends BEAR_Base
{
    pritave $_file;

   // イーガー
    public function onInject()
    {
        $this->_file = BEAR::dependency('File_Access', $config);
    }

   // 遅延
    public function onInjectLazy()
    {
        $this->_file =  $config;
    }

   // 遅延（他のクラス）
    public function onInjectLazy()
    {
        BEAR::set('file', array('My_Remote_File2', $config));
        $this->_file = 'file';
    }

    //モック
    public function onInjectMock($path)
    {
        $this->_file = BEAR::dependency('File_Access_Mock', $config);
    }

    public function onInit(array $args)
    {
        if ($maybe) {
            $file = BEAR::dependency('My_File', $this->_file);
            $file->load($path);
        }
    }

}
```


## $configの多態性 ##

```
this->_file = BEAR::dependency($class, $config);
```

$classがクラスヒント、$configが設定です。設定の変数の型に応じて動作が変わります。

  1. 配列だと、配列を「設定`=$config`」としてクラスヒントをクラス名としてインスタンスをシングルトン生成します。設定app.ymlで`__class`が指定されている時はそのクラス名が使われます。
  1. 文字列だと、それをサービスキーとしてレジストリからオブクジェトを取り出します。
  1. オブジェクトだと、そのまま使用します。（その場合クラスヒントに強制力はありません）


## 永続化オブジェクト ##
オブジェクト生成コストを削減するためにオブジェクトパーシステンシーオプションが利用できます。このオプションで生成したオブジェクトはインジェクター実行直後のオブジェクトがキャッシュされます。2回目の実行からキャッシュクリアされるまで有効です。絵文字マップ等の大きなデータやCSVのパースなどのインジェクトコストをほぼ0近いものにします。

```
$options = array('persistent' => 1);
$cache = BEAR::dependency('App_Emoji', $config, options);
```

## デフォルト値 ##

Cacheクラスはfactoryメソッドを持ち、$adapter変数によってファイルキャッシュ（1)かMemcacheクラス(2)オブジェクトをfactoryメソッドが返すとします。

```
$config = array('adapter' => 1);
$cache = BEAR::dependency('Cache', $config);
```

上記で$configを無指定にすると...
```
$cache = BEAR::dependency('Cache');
```

app.ymlで定義された値が デフォルトとして利用されます。

app.yml
```
BEAR_Cache:
  adapter: 1
```

通常のクラス生成では初期値を引数のデフォルトとして指定しますが、そのデフォルトを"分離"してDIツールで集中管理しています。

newの場合
```
function constuctt($adapter = 初期値, $arg2 = 初期値, $arg3 = 初期値)
```

newと比較して、初期値を集中して管理できるようになりました。アプリケーションによってデフォルト値を変えたい場合でもクライント、サービスコード双方に変更の必要がありません。デフォルト値がどちらにも依存してないためです。

# ページ依存の注入 #

クラスと同じようにページの依存を考えてみましょう。例えば/user?user\_id=1というページは`$_GET['user_id']`をそれを利用してるコードから取り除き、取得から注入へと変更します。

```
$userId = $_GET['user_id'];
```

このコードをテスト確認にするためには、/user?user\_id=10等とクエリーを付加して実行しないとできません。これはクライアントが利用するもの($_GETの値)を"取得"するからです。GETクエリーを付加するのは簡単ですが、リモートサイトのデータやPOSTデータなら厄介です。_

BEARではこのようなページが依存する値を"取得"の代わりに、"注入"することができます。

```
    public function onInject()
    {
        $this->injectGet('user_id', 'id', $default);
        $this->injectArg('name', $name);
    }

    public function onTestInject()
    {
        $this->injectArg('user_id', 5);
        $this->injectArg('name', 'Kuma');
    }

    public function onInit(array $args)
    {
        $userId = $args['user_id'];
        $name = $args['kuma'];
    }
```

開発時はテストインジェクターで開発すれば、クエリーの付加など利用される側の操作は必要ありません。また利用するコードには変更がないのでuser\_idの取得法が将来変わっても利用するコード(onInit)の再テストは必要ありません。テストページがつくりやすくなります。

@requiedアノテーションでページの必須項目を指定することもできます。

```
    /**
     * 初期化
     *
     * @param array $args
     * @return void
     * 
     * @required tab
     */
    public function onInit(array $args){{{
```

このページで'tab'を注入してないと400 Bad Accessが出力されて実行することができません。


### サービス（クラス）を変更する ###

#### レジストリを使う方法 ####
アプリケーションの開始時に使用が想定されるサービスをレジストリに保管しておくことで、以降の利用で使われるサービスを規定することができます。それはアプリケーションに限りません。BEARフレームワークのサービスもほとんどがこのレジストリに登録され利用されています。フレームワークのソースを書き換えることなしに先にレジストリに登録することによってフレームワークの機能の一部をオリジナルのクラスに置き換えることが可能です。

```
@insatnce singleton
```

BEARのソースで上記のようにphpdocで@instanceがsingletonなっているものがレジストリを使用していて入れ替え可能です。


例 `BEAR_Agent_Adaptor_Iphone`を自作の`App_Agent_Adaptor_Iphone`に変える例。App.phpで遅延読み込み（レイジーロード）で登録します。

```
BEAR::init($app);
BEAR::set('BEAR_Agent_Adaptor_Iphone', array('App_Agent_Adaptor_Iphone', $config));
```

#### app.ymlで指定する方法 ####
`__class`を使うとクラスの指定ができます。

app.yml
```
BEAR_Pager:
  __class: App_Pager
```

このように指定されると以下のコードでは実際には$pagerは`App\_Pager'オブジェクトが用意されます。

```
$pager = BEAR::dependency('BEAR_Pager');
```

### インジェクターを変更する ###
injectorキーを指定するとインジェクターが変更できます。文字列でクラス内部のインジェクター、配列で外部のインジェクターが使用されます。

以下は配列で指定して外部インジェクターを利用してる例です。指定したクラスでは`BEAR_Injector_Interface`を実装して元のクラス（$object）の依存（利用）オブジェクトをプロパティとして代入しています。

app.yml
```
BEAR_Pager:
  __injector:
   - BEAR_Pager_Injector
   - inject
  pager_options:
    option1: hoge1
    option2: hoge2
```

App/Pager/Injector.php
```
class App_Pager_Injector implements BEAR_Injector_Interface
{
    /**
     *　インジェクト
     *
     * @param BEAR_Pager $object オブジェクト
     * @param array      $config 設定
     */
    public static function inject(&$object, $config)
    {
        ...
        $object->pager = Pager::factory($config['pager_options']);
    }
}
```

# リンク #
  * [マーティンファウラー　Dependency Injection パターン](http://kakutani.com/trans/fowler/injection.html)
  * http://solarphp.com/SolarPHP