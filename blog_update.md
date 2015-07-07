## 編集フォームの作成 ##
編集postフォームは新規postフォームとほとんど同じです。すでに作成した記事作成フォームを新規、編集どちらにも対応できるように変えてみましょう。$defaultsというデフォルト値が入った値を受け取ったらそれをセットします。

App/Form/Post2.php
```
<?php
class App_Form_Post2 extends BEAR_Base
{
    public function build(array $defaults = null)
    {
        $form = BEAR::factory('BEAR_Form', array('formName' => 'form'));
        if ($defaults) {
            $form->setDefaults($defaults);
        }
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
```


## 編集ページの作成 ##

`$_GET['id']`がIDのPostリソースを読み込みフォームを作成しています。Submitされバリデーションが通ればPostリソースにupdateメソッドでアクセスしています。

表示用のテンプレートは作成と同じものを利用しています。


htdocs/update.php

```
<?php

require_once 'App.php';

class Page_Update extends App_Page
{
    private $_id;

    public function onInject()
    {
        $this->_resource = BEAR::dependency('BEAR_Resource');
        $this->_header = BEAR::dependency('BEAR_Page_Header');
        $this->injectGet('id', 'id');
    }

    public function onInit(array $args)
    {
        $this->_id = $args['id'];
        $params = array(
                    'uri' => 'Post',
                    'values' => array('id' => $args['id'])
        );
        $post = $this->_resource->read($params)->getBody();
        BEAR::dependency('App_Form_Post2')->build($post);
    }

    public function onOutput()
    {
        $this->display('create.tpl');
    }

    public function onAction(array $submit)
    {
        $submit['id'] = $this->_id;
        $params = array(
            'uri' => 'Post',
            'values' => $submit
        );
        $this->_resource->update($params)->request();
        $this->_header->redirect('/index.php');
    }

}

App_Main::run('Page_Update');
```