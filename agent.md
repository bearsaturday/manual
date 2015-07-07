# 導入 #

"ユーザーエージェント・スニッフィングとは、特定のユーザーエージェントで閲覧すると異なった内容を示すウェブサイトを指します"

[ユーザーエージェント](http://ja.wikipedia.org/wiki/%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E3%82%A8%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%B3%E3%83%88)

同じURLでPC・携帯でコンテンツを出し分けたりする機能です。Webの入出力(絵文字や文字コード）を変更、専用テンプレートの抽出、などが行われます。

UAコードの判別要求はアプリケーションによって様々です。例えばPCと携帯３キャリアをサポートするのか、加えてiPhoneやiPodの表示も提供するのか、iPhoneとAndroidをSmartフォンとして一括して管理するか、iPhone用のページが用意されてなかったら表示するのは携帯用テンプレートなのかPC用テンプレートなのか。

BEARはこれら様々な要求に「UAコード」、「UAの判別ロジックのインジェクト」とロールに継承関係をもった「UAアダプター」で応えます。サポートする全てのUAのテンプレートやインジェクターを使う必要はありません。「用意されていれば」使われます。例えばブログでトップページや記事のページだけiPhoneのテンプレートを用意するということがPageの変更なしにテンプレートの追加だけで実現できます。

これはViewだけに限りません、DIツールのオプションと組み合わせてエージェントロールによるインジェクタを利用することができます。クラスのUA依存が利用コードの変更なしで解決できます。

# 詳細 #

## UAコード ##
UAスニッフィングの基本になるのがユーザーエージェント(UA)コードです。UAを区別する基本IDとして、テンプレート名やインジェクタ名に使われます。

最も基本となるコードは"Default"で通常PCを想定します。大文字始まりの英単語です。BEARが用意していないコードも扱えます。以下はBEARが標準で持つコードです。

  1. Default
  1. Mobile
  1. Docomo
  1. Au
  1. Softbank
  1. iPhone
  1. iPad
  1. Android
  1. Willcom

## UAの判別 ##

アプリケーション独自のUAコードの判別を行うにはBEAR\_Agentの`$config['inject_ua']`でUA判別のインジェクタークラスを指定します。デフォルトでは`BEAR_Aget_Ua`クラスがUAの判別をしてインジェクトをしています。

以下はUA判別を独自のApp\_Agent\_Uaクラスで行う例です。

app.yml
```
BEAR_Agent:
  ua_inject: 'App_Agent_Ua'
```

以下はiPhoneかそれ以外(Default)だけを判別する単純な例です。

```
class App_Agent_Ua implements BEAR_Injector_Interface
{
    /**
     * UAインジェクト
     *
     * @param BEAR_Agent $object BEAR_Agentオブジェクト
     * @param array     $config 設定
     */
    public static function inject(&$object, $config)
    {
        if (strpos($config['user_agent'], 'iPhone') !== false) {
            // iPhoneの場合
            $ua = BEAR_Agent::UA_IPHONE;
        } else {
            $ua = BEAR_Agent::UA_DEFAULT;
        }
        $object->setService('_ua', $ua);
    }
}
```

## ユーザーエージェント・スニッフィングを有効にする ##

App/Main.phpで外部インジェクターをコールします。
```
    public function onInject()
    {
        // ユーザーエージェント・スニッフィングを有効にする
        BEAR_Main_Ua_Injector::inject($this, $this->_config);
        parent::onInject();
    }

```

## UAコード（エージェントロール） ##

UAコードとはUAをアプリケーションでの取り扱いを区別するためのエージェントのコードです。UAコードは”エージェントロール”というエージェントの役割の継承機能を持たす機能に使われます。テンプレート選択の最適化、インジェクターの最適化に使われます。

たとえばDocomoというエージェントは

Docomo > Mobile > Default

というロールの継承関係を持ちます。UAコードはデバイスに限りません、ブラウザのバージョンのロールを設定することもできます。以下は継承関係のサンプルです。

  * Docomo2010 > Docomo > Mobile > Default
  * IE7 > IE > Default
  * iPhone > Apple > Default
  * iPad > Apple > Default

このように様々なアプリケーション要求に応じてコンテンツを出し分けたり、依存性の注入を行うことができます。

## エージェントアダプター ##

それぞれのエージェントアダプターは`BEAR_Agent_Adapotr_{UAコード}`クラスに配置されます。
エージェントアダプター内では表示に関する依存やエージェントロールの設定をします。

```
 * @config bool   enable_js         JS可？
 * @config bool   enable_css        CSS可？
 * @config bool   enable_inline_css DocomoのCSS用にtoInlineCSSDoCoMo使用？
 * @config string role              ロール
 * @config array  header            HTTPヘッダー
 * @config bool   agent_filter      フィルター処理?
 * @config string output_encode     出力時の文字コード
```

この設定の多くはHTMLレンダリング用のViewアダプター（現在は`BEAR_View_Adaptor_Smarty`のみ）で使用されます。
Viewアダプターはこの設定の基づいて最終出力用のHTTPリソースオブジェクトを生成します。

※自分の想定したUA別の表示機能が`BEAR_View_Adaptor_Smarty`にないときは自作のものを作成して予めレジストリに`BEAR_View_Adaptor_Smarty`サービスとして登録します。後述のアダプターをご覧になってください。

以下はiPhoneアダプターの内容です。

```
class BEAR_Agent_Adaptor_Iphone extends BEAR_Agent_Adaptor_Mobile implements BEAR_Agent_Adaptor_Interface
{
    /**
     * コンストラクタ
     *
     * @param array $config 設定
     */
    public function __construct(array $config)
    {
        parent::__construct($config);
        $contentType = isset($this->_config['content_type']) ? $this->_config['content_type'] : 'application/xhtml+xml' ;
        $this->_config['agent_filter'] = true;
        $this->_config['header'] ='Content-Type: ' . $contentType . '; charset=utf-8;';
        $this->_config['charset'] ='utf-8';
        $this->_config['enable_js'] = true;
        $this->_config['role'] = array(BEAR_Agent::UA_IPHONE, BEAR_Agent::UA_MOBILE, BEAR_Agent::UA_DEFAULT);
    }
}
```
例えば上記ではヘッダーのコンテントタイプがデフォルトではapplication/xhtml+xmlですが、アプリケーション設定の値で変更できます。以下はHTML5用にtext/htmlにする例です。

app.yml
```
BEAR_Agent_Adaptor_Iphone:
  content_type: 'text/html'
```

### エージェントアダプターの作成、変更 ###

既存のエージェントアダプターを変更する場合（たとえばエージェントロール）や、新規のアダプターを作成するときに`BEAR_Agent_Adaptor`内のファイルを作成したり、変更したりする必要はありません。あらかじめグローバルレジストリに登録することでBEAR::dependency()はそのサービス（オブジェクト）を使用します。

例えばSmartフォン用にSmartというBEAR標準にないUAコードを考えます。App.phpなどでレジストリに「Smartアダプターサービス」を下記のようにして注入します。

```
$smart = BEAR::factory('App_Agent_Adaptor_Smart');
BEAR::set('BEAR_Agent_Adaptor_Smart', $smart);
```

これはイーガー（即時）セットの場合です。通常はレイジーセットでいいでしょう。


```
$smart = array(('App_Agent_Adaptor_Smart', array());
BEAR::set('BEAR_Agent_Adaptor_Smart', $smart);
```

サービス（オブジェクト）ではなく、サービスの生成法がレジストリにセットされ利用するときにインスタンス化されます。

上記のUAインジェクトでSmartである条件を判断して、Smartフォンと判断されたものにSmartというUAコードを注入（セット）します。既存アダプターの変更も同様です。

## UAテンプレート ##

Docomo > Mobile > Defaultというロールの場合、テンプレートは以下の順でテンプレートファイルが優先され使用されます。

  1. index.docomo.tpl
  1. index.mobile.tpl
  1. index.tpl

※レイアウトファイルも同様です。
※v0.8.69よりリソーステンプレートも同様のルールが適用されるようになりました。

## UAインジェクター ##

エージェント別のインジェクターが使用されます。

```
BEAR::dependency('App_Form_User', array(), array('injector'=>array('BEAR_Agent_Injector', 'inject')))->build();
```

以下の順でインジェクターメソッドが優先され実行されます。

  1. onInjectDocomo()
  1. onInjectMobile()
  1. onInject()

上記はapp.ymlのインジェクターの指定等でも同じです。

```
App_Form_User:
  __injector:
   - BEAR_Agent_Injector
   - inject
```
この場合は下記のようにインジェクターの指定をしなくても上記のインジェクター指定と同様の動きになります。
```
BEAR::dependency('App_Form_User')->build();
```