# 導入 #

BEARのセッションは[PEAR::HTTP\_Session2()](http://pear.php.net/manual/en/package.http.http-session2.php)を利用しています。ページにインジェクトして使います。

## $config ##

| キー| 型 | 意味　| 必須？ | デフォルト |
|:--|:--|:---|:----|:------|
| adapter | mixed | アダプター |     | 1     |
| prefix | string | セッションキープリフィックス |     | null  |
| path | string | path: dsn | session file path | memcache host(s) |     |       |
| idle | int | アイドル時間 |     |       |
| callback | string | idleタイムアウトコールバック|     |       |
| expire | int | エクスパイア時間 |     | 0     |
| expire\_callback | string | expireタイムアウトコールバック|     |       |
| gc\_max\_lifetime | int | GCが実行されない最大時間|     |       |

※ アダプター 0 なし | 1 File | 2 DB | 3 memchache

# 動作 #

App.phpでセッションエンジンを選択します。memchaced, DB, fileセッションが使用できます。※fileセッションではwebの分散化が利用できません。基本機能は以下の通りです。

```
$this->_session = BEAR::dependency('BEAR_Session');
```

```
//読み込み ２番目の引数がキーが存在しない場合のデフォルト値です
$name = $this->_session->get('user_name', '名無し');
```

```
//書き込み
$this->_session->set('cart', $cart);
```

```
//消去
$this->_session->unregister('cart');
```


その他の[HTTP\_Session2](http://pear.php.net/package/HTTP_Session2/docs/latest/HTTP_Session2/HTTP_Session2.html)の機能が利用できます。時間が経ってパスワードの入力を再度
求めたりする機能が実装できます。

## タイムアウト ##

セッションタイムアウトは２種類あります。セッション開始から最後の操作からの時間がカウントされるidleと、セッション開始してからの時間がカウントされるexpireです

### idle ###
最後のweb閲覧から一定時間たったらセッション切れが通知されるオプションです。idleでは時間を設定してcallbackでアイドル時間が切れた時にコールバックされるメソッド名を指定します。例えば10分アクセスが行われない`App_Session::onSessionTimeout()`をコールするときは以下のように設定します。
```
BEAR_Session
  idle: 3600
  callback:
    - 'App_Session'
    - 'onSessionTimeout'
```
`App_Page::onSessionTimeout()`内ではセッションタイムアウトを知らせたり、再ログインを受け付けるフォームを表示したりします。※以下のように明示的に破壊するか、expireで指定した時間がこない限りセッションデータは破壊されません。
```
  BEAR::dependency('BEAR_Session')->destory();
```

コールバック先ではセッションを破棄して無効にするか、延長して使用を続けるかを記述することができます。コールバックを指定していないとセッションは破棄されます。

### expire ###

セッション開始してからセッションタイムアウトはexpireで指定します。expire\_callbackオプションが指定されてないとこの時間のたったセッションは破棄されます。

```
BEAR_Session
  expire: 36000
```

## セッションタイムアウトサンプル ##
```
class App_Session extends BEAR_Base
{
    public function onInject()
    {
        //現在のページ
        $this->_page = $this->_config['page'];
        $this->_session = BEAR::dependency('BEAR_Session');
    }

    public function onSessionTimeOut()
    {
        // 延長
        $this->_session->updateIdle();
        // または破壊
        // $this->_session->destroy();
        //セッションタイムアウト画面
        $this->_page->display('/session/timeout.tpl');
        $this->_page->end();
    }
}
```
## DBセッション ##
DBセッションを利用する場合、デフォルトで以下のテーブルを作成する必要があります。
```
CREATE TABLE `sessiondata` (
  `id` varchar(32) NOT NULL,
  `expiry` int(10) default NULL,
  `data` text,
  PRIMARY KEY  (`id`)
) ENGINE=innodb DEFAULT CHARSET=utf8;
```CREATE TABLE `sessiondata` (
  `id` varchar(32) NOT NULL,
  `expiry` int(10) default NULL,
  `data` text,
  PRIMARY KEY  (`id`)
) ENGINE=innodb DEFAULT CHARSET=utf8;
}}}```