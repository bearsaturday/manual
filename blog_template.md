blogチュートリアル(4) テンプレートの作成

前のステップでページにPostリソース状態がセットされました。今度はリソースの表現のためのテンプレートを作成します。

## リソース用テンプレート ##
App/views/elements/post.tpl
```
<table>
    <tr>
        <td>Id</td>
        <td>Title</td>
        <td>CreatedAt</td>
    </tr>
    <!-- ここから、posts配列をループして、投稿記事の情報を表示 -->
    {foreach from=$body item=post}
    <tr>
        <td>{$post.id}</td>
        <td><a href="item.php/?id={$post.id}">{$post.title}</a></td>
        <td>{$post.created|date_format:"%Y/%m/%d %H:%M"}</td>
    </tr>
    {foreachelse}
    <tr>
        <td colspan="3">No posts found</td>
    </tr>
    {/foreach}
</table>
```

リソーステンプレートはリソースに関わらず{$code}, {$header}, {$body}の３つがアサインされます。

## ページ用テンプレート ##
App/views/page/index.tpl
```
<h1>Blog posts</h1>
{$post}
```

ページにリソースの値がそのままセットされてないことに注目してください。リソース状態（連想配列変数）にリソーステンプレートが適用されたリソース表現（HTML文字列）がプレースフォルダ{$post}に展開されます。

つまりリソースの内部構造がページテンプレートでは暴露されていません。リソース自身に適用されるリソーステンプレートの範囲で閉じ込められているので内部構造が変わってもページテンプレートに変化はありません。リソースの表現はページではなくリソースが責任を持ちます。

Note:

> リソーステンプレートをUA別に用意すれば、それぞれ異なるリソース表現が可能です。これはコーディングの変更なしにファイルの設置だけで可能です。

Note:
> リソースがレイジーセットされた場合はこの{$post}が出現するまで実リソースリクエストは行われません。例えば{if}{/if}で囲まれて出現しなかったリソースリクエストコストはかかりません。この事はviewロジック（例えば状態に応じてページの見え方が変わるなど）を可能な限りテンプレート側に持たす事を意味します。テンプレートの都合をコントローラーであるページが関心をもつ必要が低減されます。

## ページ用メタファイル ##
ページ用テンプレートのファイル名の拡張子がymlになっているファイルが”あれば”そのファイルがページ用メタインフォファイルとして扱えます。

ページ用メタインフォファイルは2ステップビュー（レイアウトファイルを使用した二段階のテンプレートレンダリング）に必要です。どのレイアウトを使うかをlayoutで指定します。

スタティックな文字列のセットにも使用します。下記の例ではtitleを指定しています。テンプレート側では{$layout.title}と指定します。

App/views/pages/index.yml
```
default:
  title: BEAR Blog
layout:default.tpl
```


## 記事ページの設置 ##

  * これまでのまとめとして「個別の記事ページ」を作成します。ページクラス、リソーステンプレート、ページテンプレート、ページテンプレート設定ファイルです。


htdocs/item.php
```
<?php

require_once 'App.php';

class Page_Index extends App_Page
{
    public function onInject()
    {
        $this->_resource = BEAR::dependency('BEAR_Resource');
        $this->injectGet('id', 'id', null);
    }

    /**
     * @required id
     */
    public function onInit(array $args)
    {
        $params = array(
            'uri' => 'Post',
            'values' => $args,
            'options' => array('template' => 'item')
        );
        $this->_resource->read($params)->set('post');
    }

    public function onOutput()
    {
        $this->display('item.tpl');
    }
}

App_Main::run('Page_Index');
```

App/views/elements/item.tpl
```
<h1>{$body.title}</h1>
<p><small>Created: {$body.created|date_format:"%Y/%m/%d %H:%m"}</small></p>
<p>{$body.body|nl2br}</p>
```

App/views/pages/item.tpl
```
{$post}
<a href="/index.php">一覧に戻る</a>
```
App/views/pages/item.yml
```
layout:default.tpl
```

Note:

> @requiredアノテーションは必須の変数を表しています。なければこのページは実行されず、メソッド内で変数の有無のチェックの必要はありません。

Note:

> MVCフレームワークではブログ記事の個別ページは１コントローラーの別アクションとして実装されています。BEARでも１ページクラス内でまとめることも可能ですが、分かりやすい理解のためにここではまとめとして別のページクラスで実装しています。

## 関連項目 ##
  * [BEAR\_Page::onOutout](onOutput.md)
  * [Smarty](http://www.smarty.net/docsv2/ja/)