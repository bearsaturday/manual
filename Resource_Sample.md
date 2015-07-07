# 導入 #

アプリケーションリソースのユーザーリソースサンプルコードです。
このユーザーリソースでは、CRUD操作全てに対応していて、プロフィールリソースへのリンクも持ちます。リンクはリソース内でカプセル化されているのでリソースクライアントからはユーザーリソースとプロフィールリソースの関係を知る事なしに、リンクで２つのリソースが接続されています。

# 詳細 #

DBオブジェクトはURIとCRUDに基づいて親クラス`App_Ro.php`でインジェクトされます。ユーザーリソースのファイルであるApp/ro/User.phpではDBオブジェクトを取得するコードがありません。ここではDBアクセスにPEAR::MDB2を用いてますが、本来Roリソースは特定のデータベースオブジェクトに依存していません。

URIとメソッドに基づいてDBの接続先がとクエリーツールがインジェクトされています。DBのSQL操作を行うBEAR\_Queryクエリーツールを使ったSELECTでは、外部からLIMITやDB Pagerの適用といった操作が行えます。クエリーを行うイベントハンドラメソッド(onRead等）では、DBの接続やテーブルの指定，ページャーオプションの指定等は行われていない事に注目してください。それらは全て対応した設定($config)で生成されたBEAR\_Queryツールが行います。

またそのBEAR\_Queryも、同じインターフェイスを実装した別オブジェクトにインジェクトができます。

## リンク ##
ユーザーリソースはプロファイルリソース、フォローリソース(to, from)とリンクしています。ユーザーリソースを取得したページ内のクライアントはその中身を知らずに他のリソースの情報が取得できます。webのリンクの仕組みと同じです。

ページ内クライアントコード
```
$resource->read($userParam)->link('profile');
```
複数のリンクもかけます
```
$resource->read($userParam)->link(array('profile', 'follow_to', 'follow_from'));
```
そこからのリンクは最後のリソースに対して行われます
```
$resource->read($userParam)->link(array('profile', 'follow_to', 'follow_from'))->('detail');
```

## DBページャー ##
通常のSELECT文がクライアントのコード変更なしでDBページャークエリーに変更できます。LIMITの指定もできます。

## トランザクション ##
onUpdateではAOPを使ったトランザクションが適用されています。トランザクションはphpdoc部分に以下のように記述します。
```
@aspect around App_Aspect_Transaction
```

## ページからのオプション ##
画面一枚あたりのアイテム数等をリソースに依存させるのではなく、ページから伝えたいときがあります。その場合はページからのオプションreadの引数の`$params['option']`を以下のようにROで取得してクエリーツールを生成します。

```
$option = $this->_config['option'];
```

## 返り値 ##

返り値はoオブジェクトに変換されページに帰ります。ユーザー一覧$userを連想配列でreturnすると以下のようなRoオブジェクトとなりページに渡されます。

  * コード 200 (OK)
  * ヘッダー なし
  * ボディ $user

UpdateとDeleteに@requiedアノテーションが指定されていて、アクセスするキーに`id`が無いとエラーリソースオブジェクト(Ro)がページに帰ります。


# ソースコード #
## `App_Ro.php` ##

URIとCRUDメソッドに基づいてデータベース接続先を決定し接続したDBオブジェクトをインジェクトします。スケールアウトの時にクライアントコードを書きなす事なくDB接続先が変えられます。セレクトのときはDBページャー可能なクエリーツールを準備、更新系メソッドの時はSQL生成のモジュールを読み込んでいます。可能な限りの準備を集中して行い、それを元に各々のインジェクターでクエリーツールをインジェクトしているのでイベントハンドラ内のDB操作は極小化されていますが、SQL操作の自由は保持しています。

```
<?php
/**
 * @APP@
 *
 * PHP versions 5
 *
 * @category   BEAR
 * @package    App
 * @subpackage App_Ro
 * @author     $Author: anonymous $ <anonymous@example.com>
 * @version    SVN: Release: $Id:$
 */

/**
 * Appリソースオブジェクト
 *
 * @category   BEAR
 * @package    App
 * @subpackage App_Ro
 * @author     $Author: anonymous $ <anonymous@example.com>
 * @copyright  anonymous All rights reserved.
 * @version    SVN: Release: $Id:$
 */
class App_Ro extends BEAR_Ro
{
    /**
     * DAO
     *
     * @var BEAR_MDB2
     */
    protected $_db;

    /**
     * インジェクタ
     *
     * <pre>
     * 操作によってDBオブジェクトを変更します
     *
     * read操作はdsnをslaveに、DBページャーを利用可能に。
     * その他更新系操作はdsnをdefaultに、トランザクション可能にしExtendedモジュール読み込みます
     * </pre>
     */
    public function onInject()
    {
        $app = BEAR::get('app');
        assert(is_string($app['App_Db']['dsn']['default']));
        assert(is_string($app['App_Db']['dsn']['slave']));
        assert(isset($this->_config['method']));
        $options['default_table_type'] = 'INNODB';
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
     */
    public function getDb()
    {
        return $this->_db;
    }
}
```

## App/Ro/User.php ##
### ユーザーリソースオブジェクト ###
```
<?php
/**
 * App_Ro
 *
 * @package    App
 * @subpackage App_Ro
 */

/**
 * ユーザーリソース
 *
 * @category   BEAR
 * @package    App
 * @subpackage App_Ro
 * @author     $Author: anonymous $ <anonymous@example.com>
 * @version    SVN: Release: $Id:$
 */
class App_Ro_User extends App_Ro
{

    /**
     * インジェクタ
     */
    public function onInject()
    {
        parent::onInject();
        $this->_queryConfig['pager'] = 1; // 1だとDBページャー 0だとLIMITクエリー
        $this->_queryConfig['perPage'] = 10; 
        $this->_query = BEAR::dependency('BEAR_Query', $this->_queryConfig, false);
    }

    /**
     * リソース作成
     *
     * @required user_name
     * @required email
     * @required password
     */
    public function onCreate($values)
    {
        $values['created_at'] = _BEAR_DATETIME; //現在時刻
        $result = $this->_query->insert($values);
        if ($this->_query->isError($result, MDB2_ERROR_CONSTRAINT)) {
            throw new Panda_Exception('IDが重複しています', 409);
        } elseif ($this->_query->isError($result)) {
            throw new Panda_Exception('登録できませんでした', 500);
        }
        return $result;
    }

    /**
     * リソース変更
     *
     * @required id
     * @required user
     * @optional profile
     *
     * @aspect around App_Aspect_Transaction
     */
    public function onUpdate($values)
    {
        $params = $values['user'];
        $params['updated_at'] = _BEAR_DATETIME;
        $where = 'id = ' . $this->_query->quote($values['id'], 'integer');
        $result = $this->_query->update($params, $where);
        if (!isset($values['profile'])) {
            return $result;
        }
        // profile
        $params = $values['profile'];
        $where = 'user_id = ' . $this->_query->quote($values['id'], 'integer');
        $params['updated_at'] = _BEAR_DATETIME;
        $result = $this->_query->update($params, $where, App_Ro::TABLE_PROFILE);
        if (!$result) {
            // updateできなかったらinsert
            unset($params['updated_at']);
            $params['user_id'] = $this->_query->quote($values['id'], 'integer');
            $$params['created_at'] = _BEAR_DATETIME;
            $result = $this->_query->insert($params, App_Ro::TABLE_PROFILE);
        }
        return $result;
    }

    /**
     * リソース読み込み
     *
     */
    public function onRead($values)
    {
        //db
        $where = isset($values['id']) ? ' WHERE id = ' . $this->_query->quote($values['id'], 'integer') : "";
        $sql = "SELECT * FROM {$this->_table}{$where}";
        if (isset($values['id'])) {
            // _db(MDB2オブジェクト）でセレクト
            $result = $this->_db->queryRow($sql);
        } else {
            // DBページャーを使うために_query(BEAR_Queryオブジェクト）でセレクト
            $result = $this->_query->select($sql);
        }
        return $result;
    }

    /**
     * リソース削除
     *
     * @required id
     */
    public function onDelete($values)
    {
        $values['deleted_at'] = _BEAR_DATETIME;
        $where = 'id = ' . $this->_query->quote($values['id'], 'integer');
        $result = $this->_query->update($values, $where);
        return $result;
    }

    /**
     * リンク
     *
     * @return array
     *
     * @required id
     */
    function onLink($values)
    {
    	$links = array();
        $links['profile'] = "user/profile?user_id={$values['id']}";
        $links['entry_latest'] = "entry/user/latest?user_id={$values['id']}";
        $links['follow_to'] = "user/follower/to?user_id={$values['id']}";
        $links['follow_from'] = "user/follower/from?user_id={$values['id']}";
        return $links;
    }
}
```