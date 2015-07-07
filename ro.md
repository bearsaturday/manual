# 導入 #

ページなどのリソースクライアントからのリソースアクセスの振る舞いを記述できるのがRo（リソースオブジェクト）クラスです。RoはCRUDやリンクのインターフェイスを備えることができます。また、リソースクライアントからのリソースをアクセスして帰ってくる結果もこのRoです。

# 詳細 #

個別リソースのRoクラスは、アプリケーション共通の`App_Ro`を継承して定義します。
例えば記事(Blog/Entry)リソースのクラス宣言は以下の通りです。命名規則とファイルの配置は他のBEARクラスと同様に、各単語が大文字始まりの`_`（アンダースコア）区切りのクラス名にします。

```
App_Ro_Blog_Entry extends  App_Ro
{
 ...
}
```

## `App_Ro` ##

`App_Ro`はアプリケーション共通でCRUDインターフェイスメソッド内で使われるサービスの設定などをします。

### `App_Ro`サンプル ###

```
<?php

class App_Ro extends BEAR_Ro
{

    /**
     * DB
     *
     * @var BEAR_MDB2
     */
    protected $_db;

    /**
     * コンストラクタ
     *
     * @param array $config 設定
     */
    public function __construct(array $config)
    {
        parent::__construct($config);
    }

    /**
     * インジェクタ
     *
     * <pre>
     * 操作によってDBオブジェクトを変更します
     *
     * read操作はdsnをslaveに、DBページャーを利用可能に。
     * その他操作はdsnをdefaultに、トランザクション可能にしExtendedモジュール読み込みます
     * </pre>
     */
    public function onInject()
    {
        $app = BEAR::get('app');
        assert(is_string($app['App_Db']['dsn']['default']));
        assert(is_string($app['App_Db']['dsn']['slave']));
        assert(isset($this->_config['method']));
        //$options['default_table_type'] = 'INNODB';
        if ($this->_config['method'] === 'read') {
            $dsn = $app['App_Db']['dsn']['slave'];
            $config = array('dsn' => $dsn, 'options' => $options);
            $this->_db = BEAR::factory('BEAR_Mdb2', $config);
            $this->_queryConfig =  array('db'=>&$this->_db, 'ro'=>&$this, 'table'=>$this->_table, 'pager'=>0, 'perPage'=>10, 'options'=>array('accesskey'=>true));
        } else {
            $dsn = $app['App_Db']['dsn']['default'];
            $options['use_transactions'] = true;
            $config = array('dsn' => $dsn, 'options' => $options);
            $this->_db = BEAR::factory('BEAR_Mdb2', $config);
            $this->_db->loadModule('Extended');
            $this->_queryConfig =  array('db'=>&$this->_db, 'ro'=>&$this, 'table'=>$this->_table);
        }
        // すべてのフィールド識別子が SQL 文中で自動的にクォート
        $this->_db->setOption('quote_identifier', true);
    }

    /**
     * インジェクタ
     *
     * SELECTクエリーをCOUNTに変更します
     *
     */
    public function onInjectCount()
    {
        $this->onInject();
        $this->_query = BEAR::dependency('BEAR_Query_Count', $this->_queryConfig);
    }

    /**
     * DAO取得
     *
     * AOP用。トランザクションアドバイスがDBオブジェクトを取得するのに使用しています。
     */
    public function getDb()
    {
        return $this->_db;
    }
}
```

> ※この`App_Ro`はあくまで一例です。PEAR::Mdb2を使っていますが、BEARは特定のDBライブラリの依存はありません。Zend\_DbやPDOの他にDoctrineなどのORMを使う事もできます。

> ※BEAR\_Queryは現在Mdb2に依存しています。

## 設定 ##

コンストラクタで渡される設定（$config配列）は以下の通りです。

| **意味** | **型** | **キー** |
|:-------|:------|:-------|
| crudインターフェイス | string | method |
| URI    | string | uri    |
| 引数     | array | values |
| オプション  | array | options |
| ※アプリ情報 | array | info   |
| ※デバックモード | bool  | debug  |

※全クラス共通です。

## インジェクト ##

App.phpでBEAR::init($app)とBEARの初期化をした全クラスの初期値（デフォルトではapp.ymlを読み込んだ配列）はこのように取り出せます。

```
$app = BEAR::get('app');
```

$app内のDB接続先情報をいれていおいて、コンストラクタに渡される設定に基づいてonInject()で適切なサービスを注入します。

例えばメソッド(Read or 非Read)によってSlave / Master DBを変えます。あるいはユーザーリソースでIDによるパーティショニング（DB変更）がある場合だと、uriとvaluesによって接続DBを変えることもできます。

このように`App_Db::onInject()`メソッド内でDB接続しておきます。
※MDB2のレイジー接続なので実際のDB接続はSQLが発生してからです

個別のRoクラスは`App_Ro`クラスを継承して個別のリソースを実装します。以下の様にリソースのCRUDインターフェイスを実装します。

## リソースサンプル ##

```
/**
 * エントリーリソース
 *
 */
class App_Ro_Blog_Entry extends App_Ro
{
    /**
     * インジェクト
     */
    public function onInject()
    {
        parent::onInject();
        $this->_queryConfig['pager'] = 1;  // DBページャー利用
        $this->_queryConfig['perPage'] = 5; //画面毎のアイテム数
        $this->_queryConfig['deleted_at'] = true; //論理削除対応
        $id = array('id', 'id', '+');
        $date = array('created_at', 'date', '-');
        $this->_queryConfig['sort'] = array($id, $date); //ソートクエリー可
        $this->_query = BEAR::dependency('BEAR_Query', $this->_queryConfig, false);
    }

    /**
     * リソース作成　(AOPトランザクション)
     *
     * @required user_id
     * @required body
     *
     * @aspect App_Aspect_Transaction
     */
    public function onCreate($values)
    {
        $values['created_at'] = _BEAR_DATETIME; //現在時刻
        $result = $this->_query->insert($values);
        if ($this->_query->isError($result)) {
            throw $this->_exception('登録できませんでした', 500);
        }
    }

    /**
     * リソース読み込み
     *
     * ソートとDBページャーとRow/All取得に対応します
     */
    public function onRead($values)
    {
        $sql = "SELECT * FROM entries";
        $result = $this->_query->select($sql, array(), $values);
        return $result;
    }
}
```

`App_Ro`でDB接続など基本設定された`_queryConfig`をさらに個別のリソースのインジェクタで設定（ページャーやソート可など）しています。

## 継承Ro ##

Roはクラスなので一部の機能だけが違う別リソースをつくるのがクラスの継承を用いて簡単にできます。

上記の記事を自由にページングできるBlog/Entryリソースから全てのEntryを表示するBlog/Entry/Allを考えてみます。違うのはインジェクトだけならクラスの継承で簡単に実現できます。

```
class App_Ro_Blog_Entry_All extends App_Ro_Blog_Entry
{
    /**
     * インジェクト
     */
    public function onInject()
    {
        parent::onInject();
        $this->_queryConfig['pager'] = 0; //画面毎のアイテム数
        $this->_queryConfig['perPage'] = 0; //画面毎のアイテム数
    }
}
```

## リソースリンク ##
他へのリソースリンクをonLinkメソッドに記述でき、リソース利用側からみたリソースの関係をカプセル化することができます。詳しくは[link](link.md)をご覧ください