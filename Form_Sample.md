# 導入 #

フォームのサンプルです。

また@PEAR-DIR/docs/に入っている[QuickFormのサンプルファイル](http://svn.php.net/viewvc/pear/packages/HTML_QuickForm/trunk/docs/)もご覧下さい。
特に[エレメンツ](http://svn.php.net/viewvc/pear/packages/HTML_QuickForm/trunk/docs/elements.php?view=markup)、[グループ](http://svn.php.net/viewvc/pear/packages/HTML_QuickForm/trunk/docs/groups.php?view=markup)、[ルール](http://svn.php.net/viewvc/pear/packages/HTML_QuickForm/trunk/docs/rules-builtin.php?view=markup)が参考になります。[レンダラー](http://svn.php.net/viewvc/pear/packages/HTML_QuickForm/trunk/docs/renderers/)や[テンプレート](http://svn.php.net/viewvc/pear/packages/HTML_QuickForm/trunk/docs/renderers/templates/)のサンプルもあります。

# 詳細 #

DHTMLレンダラでフォームのレイアウトを自動にしています。調整はフォームのテンプレートを変更して行います。

## テンプレート ##

レイアウトは自動なのでプレイスホルダを指定するだけです。フォーム名と同じになります。

```
{$form} {* デフォルト *}
{$formLogin} {* ログインフォーム *}
```

## フォーム作成コード ##

ファイルの場所を集中しておくと管理や利用で便利でしょう。下記はフォームのメッセージをapp.ymlに記述したフォームのサンプル例です。このように言語ファイルをconfigで準備しておく必要は必ずしもありませんが、多言語サイトが作りやすくなるのとメンテナンス性が向上します。

### App/Form/Blog/Entry.php ###

```
<?php
/**
 * 記事フォーム
 *
 * described here...
 *
 * @package    App
 * @subpackage Form
 * @author     $Author: koriyama@users.sourceforge.jp $
 * @version    SVN: Release: $Id:$
 */
class App_Form_Blog_Entry extends BEAR_Base
{

    /**
     * フォーム設定
     *
     * @var array
     */
    private $_form = array('formName' => 'form');

    /**
     * フォームアトリビュート
     *
     * @var array
     */
    private $_attr = array('title' => 'size="30" maxlength="30"',  'body'=>'rows="8" cols="40"');

    /**
     * コンストラクタ
     *
     * @param array $config
     */
    public function __construct(array $config)
    {
        parent::__construct($config);
    }

    /**
     * インジェクト
     *
     */
    public function onInject()
    {
        parent::onInject();
    }

    /**
     * フォーム
     *
     * @return void
     */
    public function build()
    {
        $form = BEAR::factory('BEAR_Form', $this->_form);
        // ヘッダー
        $form->addElement('header', 'main', $this->_config['label']['main']);
        // フィールド
        $form->addElement('text', 'title', $this->_config['label']['title'], $this->_attr['title']);
        $form->addElement('textarea', 'body', $this->_config['label']['body'],  $this->_attr['body']);
        $form->addElement('submit', '_submit', $this->_config['label']['submit'], '');
        // フィルタと検証ルール
        $form->applyFilter('__ALL__', 'trim');
        $form->applyFilter('__ALL__', 'strip_tags');
        $form->addRule('title', $this->_config['error']['title_required'], 'required', null, 'client');
    }
}
```

## app.yml ##
```
App_Form_Blog_Entry:
  label:
    main: '投稿画面'
    title: 'タイトル'
    body: '本文'
    submit: '記事を投稿'
  error:
    title_required: 'タイトルを入力してください'
```App_Form_Blog_Entry:
  label:
    main: '投稿画面'
    title: 'タイトル'
    body: '本文'
    submit: '記事を投稿'
  error:
    title_required: 'タイトルを入力してください'
}}}```