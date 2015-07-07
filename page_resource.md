# 導入 #

ページをリソースとして扱う事ができます。

ページリソース結果のタイプは２つあります。pageのhttp出力結果をリソース結果とする **html** とページ上でsetされたリソースを値として利用する **resource** の２つです。

# ページをリソースに #

参照系のページでページクラスの役割はリソースにアクセスし、そのリソース状態をviewコンポーネントでHTTP出力することです。

ページクラスではリソースをアクセスする部分とその結果を出力する部分がそれぞれ`onInit(array $args)`, `onOutput()`と分かれています。MVC的に役割をみるとページクラスはリソースというモデルを操作するコントローラーになりますが、そのページをリソースとして扱うというのがページリソースです。コントローラー自身がモデルになるともいえます。またリソースは他のリソースを呼ぶ事ができるというROAのレイヤーの原則をページにも適用しようとするものでもあります。

App/Ro以下に配置するRoリソースの形式とhtdocs/以下に配置するページリソースの構造は似ている事にお気づきでしょう。どちらもコンストラクタに渡されるarray $configという設定でオブジェクトが作成され、onInject()でそのクラスで必要なサービス（オブジェクト）を利用可能にし、onInit(array $args)またはonRead(array $values)で下位のデータソースレイーヤーに値を求め結果をsetまたはreturnします、どちらもオブジェクトを利用するのに専用のクラスを用いユーザーが直接オブジェクトを扱う事はありません。


### Roクラス ###
```
class App_Ro_Entry extends App_Ro
{
    public function __construct(array $config)
    {
        parent::__construct($config);
    }

    public function onInject()
    {
        ....
    }

    public function onRead($values)
    {
        ...
        return $entry;
    }

    public function onUpdate($values)
    {
        ...
    }
```

### ページクラス ###
```
class Page_Entry extends App_Page
{
    public function __construct(array $config)
    {
        parent::__construct($config);
    }

    public function onInject()
    {
        ....
    }

    public function onInit(array $args)
    {
        ...
        $this->_resource->read($params)->set('entry');
    }

    public function onOutput()
    {
        $this->display();
    }

    public function onAction(array $submit)
    {
    ....
    }
}
```

ページクラスをRoのように扱うのがページクラスです。
上記のページクラスをread操作したときはonInit(arra $args)が呼ばれ、read以外（区別はありません）の操作をしたときにはonAction(array $submit)が呼ばれます。引数はどちらも同じで'values'パラメーターで指定する連想配列が１つだけ渡ります。リソース結果はオプションの指定によって異なります。

### uriと引数 ###

#### read ####
| **uri** | **values** |
|:--------|:-----------|
| ページスクリプトパス | onInit(array $args)の$args|


#### create, update, delete ####
| **uri** | **values** |
|:--------|:-----------|
| ページスクリプトパス | onAction(array $submit)の$submit |

ページスクリプトパスは拡張子(php)を除いたものを使います。

例）
htdocs/db/select/item.phpならURIは\*page://db/select/item**になります**


### オプション ###

#### resourceオプション ####
onInit()内でsetしたリソースの値を連想配列にしたものをボディにしたリソースオブジェクトが返ります。
```
        $uri = 'page://db/select/item';
        $values = array('id' => 1);
        $options = array('output' => 'resource');
        $params = array('uri' => $uri, 'values' => $values, 'options'=>$options);
        $this->_resource->read($params)->set('entry');
```

#### htmlオプション ####

onOutput()内のdisplay()で主力されるHTTPのリソースオブジェクトが返されます。HTTPのヘッダーがリソースオブジェクトのヘッダー、HTTPボディがリソースオブジェクトのボディになります。

```
        $uri = 'page://db/select/item';
        $values = array('id' => 1);
        // UAコードDocomoでDocomo用ページをアクセスしたHTTP結果を文字列(=html)として取得
        $options = array('output' => 'html', 'page' => array('ua' => 'Docomo'));
        $params = array('uri' => $uri, 'values' => $values, 'options' => $options);
        $html = $this->_resource->read($params)->getBody();
```

## Link ##

