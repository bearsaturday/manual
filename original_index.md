BEARはリソース指向のPHPアプリケーションフレームワークです。

 * これはPHP5.2+対応の[BEAR.Saturday](https://github.com/koriym/BEAR.Saturday)です。
 * PHP5.5+に対応したメジャーバージョンアップ版は[BEAR.Sunday](http://bearsunday.github.io/ )です。

http://www.youtube.com/watch?v=NKdiNdNbH0Y

ページ

```php
<?php

require 'App.php';

/**
 * 記事一覧表示ページ
 *
 * @package Page
 * @author  $Author:$
 * @version SVN: Release: $Id: $
 */
class Page_User_Entry extends App_Page
{

    /**
     * インジェクト
     *
     */
    public function onInject()
    {
        $this->_resource = BEAR::dependency('BEAR_Resource');
        $this->injectGet('id');
    }

    /**
     * 初期化
     *
     * @requied id
     */
    public function onInit(array $args)
    {
        $params = array('uri' => 'User', 'values' => $args);
        $this->_resource->read($params)->link('entry')->set('user_entry');
    }

    /**
     * 出力
     *
     */
    public function onOutput()
    {
        $this->display();
    }
}
$options = array('cache' => array('type' => 'init', 'life' => 10));
App_Main::run('Page_User_Entry', $options);
```

リソース

```php
<?php
/**
 * エントリーリソース
 *
 */
class App_Ro_Entry extends App_Ro
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
        $this->_queryConfig['sort'] = array(array('created', 'date', '+')); //ソートクエリー可
        $this->_query = BEAR::factory('BEAR_Query', $this->_queryConfig, false);
    }

    /**
     * リソース作成　(AOPトランザクション)
     *
     * @required user_id
     * @required body
     *
     * @aspect around App_Aspect_Transaction
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

リソーステンプレート
```
<a href="?_sort=id">↓古い順</a>&nbsp;<a href="?_sort=-id">↑新しい順</a>
<ul class="entry">
	{foreach item=item from=$entry}
	<li>
	   {$item.title} - {$item.body|mb_truncate:120}
	</li>
	{/foreach}
</ul>
```

ページテンプレート
```
<h2>エントリー一覧</h2>
{$user_entry}
{$pager.links.all}
```
