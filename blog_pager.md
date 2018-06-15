## ページング ##

`BEAR_Query`でリソースのページングと表示順をコントロールする機能を追加します。

リソースのreadインターフェイスの実装を憶えてますか？

```
    public function onRead($values)
    {
        $sql = "SELECT * FROM posts";
        $result = $this->_query->select($sql, array(), $values);
        return $result;
    }
```

これにページング機能を追加するために、この利用コードのSQLに\*変更はありません。**インジェクター（初期化メソッド）での設定が変わるだけです。**

App/Ro/Pager.php
```
<?php
class App_Ro_Post_Pager extends App_Ro_Post
{
    public function onInject()
    {
        parent::onInject();
        $this->_queryConfig['pager'] = 1; // DBページャー利用
        $this->_queryConfig['perPage'] = 10; // １ページ毎のアイテム数
        $id = array('id', 'id', '+');
        $date = array('created', 'date', '-');
        $this->_queryConfig['sort'] = array($id, $date); // ソート
        $this->_query = BEAR::factory('BEAR_Query', $this->_queryConfig);
    }
}
```

これでページングとソートを指定しています。ページング部分に限れば２つだけです。

```
        $this->_queryConfig['pager'] = 1; // DBページャー利用
        $this->_queryConfig['perPage'] = 10; // １ページ毎のアイテム数
```

以下のようにpager=0にするとページング機能は働かず最初の10件のリソースになります。
```
        $this->_queryConfig['pager'] = 0; // DBページャー利用
        $this->_queryConfig['perPage'] = 10; // １ページ毎のアイテム数
```

例えばブログの記事を考えています。記事リソースは場所によってリソース表現が変わります。

| **内容** | **ページング** | **pager** | **perPage** | 表示順 |
|:-------|:----------|:----------|:------------|:----|
| トップページ | 最近記事のリソースが最新3件 | 0         | 3           | -id |
| 今月の記事ページ | ページングなし   | 0         | 0           | +id |
| 記事本体ページ | DBページング   | 1         | 10          | -id |

設定（準備）が違うだけでクエリーそのものを変えずに、それぞれの表現で取得しています。このサンプルでは別クラスにしてインジェクターをオーバーライドしてますが、複数のインジェクターをもちそのインジェクターを切り替えて使う事もできます。

Note:
> `BEAR_Query`は使用されるSQLのORDER句より前の部分を本質的で固定的なもので、その後ろを表現のための流動的で装飾的なものととらえそれぞれを利用と準備で分け、再利用性を高めています。

Note:
> 件数を求めるためにCOUNTクエリーを行う時にも利用コードに変更はありません。詳しくは[db](db.md)をご覧ください。

Note:
> BEARではリソースのページングの記述はリソース側のみに閉じ込められており、ページコントローラー側に記述はありません。

## テンプレート ##
ページャーを使うと$pagerという変数がテンプレートにアサインされます。

|$pager.info | ページング情報 |
|:-----------|:--------|
|$pager.links | ページングナビゲーション |

```
{$pager.info.totalItems}件中{$pager.info.from}~{$pager.info.to}件表示
<a href="?_sort=id">↓古い順</a>&nbsp;<a href="?_sort=-id">↑新しい順{$post}
{$pager.links.all}
```

効果的なSEOのため、クローラーにページング情報を教えるためにはHTMLの`<head>`タグ内に`<link>`タグ設置します。
```
<link rel="start" href="{$pager.links.linkTagsRaw.first.url}" title="" />
<link rel="last" href="{$pager.links.linkTagsRaw.last.url}" title="" />
<link rel="next" href="{$pager.links.linkTagsRaw.next.url}" title="" />
<link rel="prev" href="{$pager.links.linkTagsRaw.prev.url}" title="" />
```

どのような変数がアサインされてるかはBEAR Dev画面を見ても確認できます。

## 他のフレームワークでは ##

ページングはフルスクラッチで実装するのになかなか面倒な機能です。フレームワークを使うとより簡単に設置できます。

  * [Symfony1 - リストのパジネーション](http://www.symfony-project.org/jobeet/1_2/Propel/ja/07)
  * [CakePHP - ページ付け(Pagination)](http://book.cakephp.org/ja/view/164/%E3%83%9A%E3%83%BC%E3%82%B8%E4%BB%98%E3%81%91-Pagination)
  * [Zend Framework - データコレクションのページ処理](http://framework.zend.com/manual/ja/zend.paginator.usage.html)
  * [Agavi - Introduction to MVC programming with Agavi, Part 5: Add paging, file uploads, and custom input validators to your Agavi application](http://www.ibm.com/developerworks/library/x-agavipt5/)
  * [Yii - ページネーション](http://d.hatena.ne.jp/tjtjtjofthedead/20110322/1300805961)

## 関連情報 ##

  * [db](db.md)