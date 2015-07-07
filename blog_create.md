これまでのステップでデータベースに登録されている記事を表示できるようになりました。次はいよいよ、フォームを作成してデータを追加・編集できるようにしてみましょう。

## 記事を追加するページの作成 ##

ページコントローラーであるBEARは"action per page"が基本です。記事の追加用のページをつくります。

create.php
```
<?php

require_once 'App.php';

class Page_Create extends App_Page
{
    public function onInject()
    {
        $this->_resource = BEAR::dependency('BEAR_Resource');
        $this->_header = BEAR::dependency('BEAR_Page_Header');
    }

    public function onInit(array $args)
    {
        BEAR::dependency('App_Form_Post')->build();
    }

    public function onOutput()
    {
        $this->display('create.tpl');
    }

    public function onAction(array $submit)
    {
        $params = array(
            'uri' => 'Post',
            'values' => $submit
        );
        $this->_resource->create($params)->request();
        $this->_header->redirect('/index.php');
    }
}

App_Main::run('Page_Create');
?>
```

このページでは閲覧ページには無かったonActionというメソッドが加わりました。このメソッドはフォームのバリデーションが取った時だけ呼ばれます。※このときonOutput()は呼ばれません。

onAction内では記事の作成のためにPostリソースがcreateメソッドでアクセスされています。その後にheaderサービスオブジェクトのredirectメソッドで記事一覧ページにリダイレクトしています。

Note:
> $submitは単にサブミットされた値というだけではなく、フォームに登録し生成時の制限にしたがった信頼のできるものです。フォームオブジェクトに加えられてないもしサブミットされてもエレメントは無視されます。

Note:
> アンダースコアで始まるエレメントは"private"に使用してるものとして$submitには渡りません。ボタン名やトークン、制御に使われるものに使います。あるいは「規約に同意」チェックはバリデーションの為には必要ですが、後のリソースリクエストには必要ありません。そのような場合にも使います。


App/Form/Post.php
```
<?php
class App_Form_Post extends BEAR_BAse
{
    public function onInject()
    {
    }

    public function build(array $defaults = array())
    {
        $form = BEAR::factory('BEAR_Form', array('formName' => 'form'));
        $form->setDefaults(defaults);
        $form->addElement('text', 'title', 'タイトル', 'size=30 maxlength=30');
        $form->addElement('textarea', 'body', '本文', 'cols=30 rows=10');
        // フィルタと検証ルール
        $form->applyFilter('__ALL__', 'trim');
        $form->applyFilter('__ALL__', 'strip_tags');
        $form->addRule('title', 'タイトルを入力してください', 'required');
        $form->addRule('body', '本文を入力してください', 'required');
        $form->addElement('submit', '_submit', 'Save Post', '');    
    }
}
?>
```

このページのビューテンプレートを設置します。

App/views/pages/create.tpl
```
<h1>New Post</h1>
{$form}
```

フォームもリソースのセットと同様に、ページテンプレート内では内部構造を暴露してないことに注目してください。フォームのエレメントをページテンプレートの変更なしに動的に変える事が用意です。またフォームに変更があったときでもページテンプレートに変更の必要がありません。UAに応じたフォームをレンダリングする事も可能です。

App/views/pages/create.ymp
```
layout:default.tpl
```

Post.phpに下記のコードを追加します
App/Ro/Post.php
```
    public function onCreate($values)
    {
        $result = $this->_query->insert($values);
    }
```



## ブラウザで確認 ##

コードの入力が完了したら、ブラウザで/create.php にアクセスしてみてください。新規追加用のフォームが表示されたら、何かデータを入力して「Save Post」ボタンをクリックし、データが正しく追加されるかどうか確認して下さい。

## 関連項目 ##

  * [form](form.md)
  * [db](db.md)
  * [PEARマニュアル QuickForm](http://pear.php.net/manual/ja/package.html.html-quickform.php)
  * [PEARマニュアル HTML\_QuickForm\_Renderer\_Tableless](http://pear.php.net/manual/ja/package.html.html-quickform-renderer-tableless.php)
  * [PEARマニュアル MDB2](http://pear.php.net/manual/ja/package.database.mdb2.php)