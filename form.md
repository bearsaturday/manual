# 導入 #

BEARではフォームの基本機能（デフォルト値、フォーマッティング、検証、再投入など）に[PEAR::HTML\_QuickForm](http://pear.php.net/manual/en/package.html.html-quickform.php)を利用します。BEARはサブミットされた文字をの絵文字、コードの変換を行ったり複数のフォームのサポート、セキュリティ機構、フォームオブジェクトの操作（生成、バリデーション、トラッキング）等を行います。

また`PEAR::HTML_QuickForm`を利用しないフォームも可能です。bear-demoの[SmartyValidateフォーム](http://code.google.com/p/bear-demo/wiki/smartyvalidate)をご覧ください。

## BEAR\_Form ##

### $config ###

| キー| 型 | 意味　| 必須？ | デフォルト |
|:--|:--|:---|:----|:------|
| adapter | mixed | フォームレンダラー |     | 0     |
| formName | string | フォーム名 |     | form  |
| method | string | POST or GET |     | post  |
| action | string | アクション |     |       |
| target | string | ターゲット |     |       |
| attributes | mixed | `<form>`タグの属性 |     | null  |
| callback | callback |レンダラーコールバック |     |       |


# 動作 #

送信されたデータはBEARがバリデーションを行い、NG,OK、それぞれ別のページメソッドに渡されます。このとき送信されたデータは送信元が携帯（SJIS）でもUTF-8に変換され絵文字はエンティティ(&63647;など)に変換されているのでページクラス内では特にクライアントのコードや文字を意識する必要はありません。

一つのページに複数のフォームがあるマルチフォームも可能です。BEARがフォーム送信のトラッキングを行い適切なページアクションメソッドがコールされます。

またBEARではフォームのレンダリングエンジンを切り替えることが可能です。デフォルトのレンダラーではフォームが自動でレイアウトされてレンダリングされます。エレメントやヘッダー毎にテンプレートが指定できます。テンプレートでは一つのテンプレート変数（`{$form}`）にすべてのフォームHTMLがレンダリングされます。

一方、フォームを個別の要素としてレンダーする方法もあります。（`{$form.title.label}{$form.title.error}{$form.title.html}`）

自動レイアウトの大きなメリットは複数エージェントの対応です。エージェントに応じたテンプレートが使用できるので、携帯もPCまたはiPhoneなども同じページテンプレートにすることができます。

フォームの作成はページのonAction()メソッドではできません。ここのメソッドにくるまでにフォームのバリデーションが行われるためです。

# セキュリティ #
## 登録保証 ##

> フォームは登録された範囲の値しか受け付けません。ラジオボタンやメニューで登録された値以外のものが送信されてもその値を無効とします。 登録されて以内フォームエレメントからの送信も無効にします。この機構は`HTML_QuickForm::exportValues()` により実装されています。

## 文字コードの保証 ##

表示しているページ以外の文字コードが送出された場合、これを不正リクエストとみなして400 Bad Requestのレスポンスが返ります。例えば携帯用にSJISで表示されているページで、悪意のあるスクリプトからUTF-8が送出されるとそれを受け付けません。

## CSRFの防止 ##

ワンタイムトークンでクロスサイトスクリプトフォージェリを防止しています。

## 二重投稿の防止 ##

  * ワンタイムトークンを使い二重投稿の防止も行えます。これは特に通信の不安定な携帯の時に有効です。（通信不良でレスポンスが届かない場合、携帯で再度リクエスト->二重投稿になるのを防ぎます）



# ページ内のフォーム数 #

ページにフォームが一つあるシングルフォームページ、二つ以上あるマルチフォームページをgetInstance時に指定します。テンプレートの書き方が少し変わってきます。

## シングルフォーム ##

フォームオブジェクトを取得します。デフォルトのフォーム名は'form'です。

```
$config = array('formName' => 'form');
$form = BEAR::factory('BEAR_Form', $config);
```

BEARFormはPEAR::HTMLQuickformを継承し日本語化など初期設定をすませたものです。 ページ内でルール、フィルターを決めてフォームを追加していきます。

```
$form->addElement('text', 'name', '名前', 'size=30 maxlength=30');
$form->addElement('text', 'email', 'メールアドレス', 'size=30 maxlength=30');
$form->addRule('name','名前を入力してください', 'required', null, '');
$form->addRule('email','emailを入力してください', 'required', null, '');
$form->addRule('email','emailの形式で入力してください', 'email', null, ''); 
$form->setJsWarnings('入力エラー', '');
$form->addElement('submit','_submit','送信');
```

サブミットしたフォームバリデーションはBEARが行います。 OKだったデータは$submitとしてonActionメソッドに渡されます。

```
class Form_Simple extends App_Page
  ...
  function onAction($submit)
  {
      //メソッド内でDBアクセスなど行います。
  }
}
```
  * $submitは $form->addElementで追加されたフォームだけが渡されます。
  * アンダースコア始まりのフォームは$submitには渡されません


## マルチフォーム ##

ページ
```
        $loginForm = BEAR::factory('BEAR_Form', array('formName' => 'login'));
        // ログインフォーム
        $loginForm->addElement('header', 'main', 'ログイン');
        $loginForm->addElement('text', 'id', 'ID', 'size=15');
        $loginForm->addElement('text', 'password', 'パスワード', 'size=15');
        $loginForm->addElement('submit', '_submit', 'ログイン', '');
        $loginForm->addRule('id', '入力してください', 'required', null, '');
        $loginForm->applyFilter('__ALL__', 'trim');
        // 確認フォーム
        $confirmForm = BEAR:: factory('BEAR_Form', array('formName' => 'confirm'));
        $confirmForm->addElement('header', 'main', 'パスワードを指定のアドレスに送信します');
        $confirmForm->addElement('text', 'email', '確認メールアドレス', 'size=30');
        $confirmForm->addElement('submit', '_submit', '確認メールアドレス送信', '');
        $confirmForm->addRule('email', '入力してください', 'required', null, '');
        $confirmForm->addRule('email', 'emailアドレスを入力してください', 'email', null, '');
        $confirmForm->applyFilter('__ALL__', 'trim');
```
マルチフォームはフォーム生成時に名前を配列で指定します。 サブミットされたデータはシングルフォームと同じくonActionメソッドにわたってからフォーム別のアクションメソッドが用意されてたらそちらもonAction()の後にコールされます
```
function onActionSearch($submit)
{
    $this->display('form/multi.search.tpl');
}

function onActionLogin($submit)
{
    $this->display('form/multi.login.tpl');
}
```
  * onAction()に共通処理をかき、onAction個別フォーム()に個別の処理を記述します
  * 個別フォーム処理のメソッドは必須ではありません。

# フォームレンダラ #

フォームレンダラとはスクリプトでのフォームオブジェクトを実際のHTMLに出力するクラスです。 デフォルトのレンダラは`HTML_QuickForm_DHTMLRulesTableless` ベースとしてBEARがテンプレートを用意するAppレンダラです。このレンダラはフォームのレイアウトを自動でおこないます。他にはSmartyレンダラがあり、こちらはフォームのレイアウトを手動で行う必要があります。


## レンダラによるテンプレートの記述の違い ##
### BEAR::RENDERER\_APPレンダラ (0) ###
```
{$formSearch}
```
フォームHTMLが集約されています。フォームレイアウトはフォームテンプレートで指定します。
### RENDERER\_SMARTY\_ARRAYレンダラ (1) ###
```
{$form.search.word.error}{$form.search.wordlabel}{$form.search.word.html}
```
フォームHTMLは部品としてアサインされます。

## HTMLQuickFormRenderer\_ArraySmartyレンダラ(0) ##

エラーテンプレートが使用できます。また必須要素指定したフォームの横にサインが出せます。デフォルトではエラーの時はラベルが赤くなり、必須要素項目は赤の＊が入力項目フォームの隣に表示されます。詳細は検証エラーが発生した要素をレンダリングする方法を設定する 、必須要素をレンダリングする方法を設定する をご覧ください。
エラーテンプレート

BEAR\_Form::$smarty\_form\_error\_templateを変更するとエラーテンプレートが変更できます。

```
//デフォルト
BEAR_Form::$error_template = '{if $error}<pre><font color="red">{$label}</font></pre>{/if}';
```

## 必須要素テンプレート ##
```
BEAR_Form::$smarty_form_required_templateを変更するとエラーテンプレートが変更できます。
```
```
//デフォルト
BEAR_Form::$required_template = <nowiki>'{$html}{if $required}<font color="red">*</font>{/if}';</nowiki>
```

## HTMLQuickFormDHTMLRulesTablelessレンダラ(1) ##

このレンダラでは`HTML_QuickForm`が作成するクライアント側の検証時の警告画面を、標準のJavaScript アラートウインドウから DHTML に置き換えます。また"onBlur" イベントや "onChange" イベントでもエラーを表示させることができます 詳細は[HTML\_QuickForm\_DHTMLRulesTableless](http://pear.php.net/package/HTML_QuickForm_DHTMLRulesTableless/docs)使用法をご覧ください。

このレンダラはコンストラクタで指定します。1ページの中で違うレンダラが混在が可能です。

```
 $form = BEAR::factory('BEAR_Form', 'adapter'=>BEAR_Form::RENDERER_SMARTY_ARRAY);
```
デフォルトは自動レイアウトのAPPレンダラです。
```
 $form = BEAR::factory('BEAR_Form', 'adapter'=>BEAR_Form::RENDERER_APP);
```


## Appレンダラ(1) ##

HTML\_QuickForm\_DHTMLRulesTablelessレンダラを継承したフォームレンダラです。
[Default.php App/Form/Default.php](http://code.google.com/p/bear-project/source/browse/BEAR/trunk/data/app/App/Form/Renderer/Default.php)で定義されたフォームが使われます。このフォームをベースとし、フォームのコンストラクタのcallbackオプションでフォームレンダラーを部分的に変更します。

## GETフォーム ##

methodオプションを指定します

```
 //デフォルト
 $options = array('method' => 'get', 'target'=>'newwin');
 $form = BEAR::factory('BEAR_Form', $options);
```

## タグののアトリビュート ##

  * 基本的には生成時に指定しますが生成してからの変更もできます。
  * 生成したタグのアトリビュートを後から変更するには以下のようにします。 {{{
```
$form->updateAttributes('enctype'=>'multipart/form-data');
```

## サブミットヘッダー ##

  * フォームの属性情報を”サブミットヘッダー”としてフォームに付け加えることができます
  * 確認ページ、アクションページなどページの出し分けや等に使います。BEARではトークンなどをサブミットヘッダーで扱っています。
  * 内部的にはフォームにアンダースコアで始まるフォームはヘッダーとみなされ、onAction($submit)として渡される$submitには含まれないものとなり、またバリデーションの対象からも外れます。

## レンダラーコールバック ##

レンダラーコールバックオプションを指定するとフォームのレンダリング時にオプションで指定されたメソッドがコールされます。メソッド内ではフォームのエレメントテンプレートを変更することができます。

以下はアダプター0（デフォルト）のフォームレンダラーのコールバックメソッドの例です。`$render`にHTML\_QuickForm\_Renderer\_Tablelessクラスのオブジェクトが渡されるので、そのオブジェクトを操作します。

```
    /**
     * レンダラーコールバック
     *
     * @param HTML_QuickForm_Renderer_Tableless $render
     *
     */
    public static function onRender($render)
    {
        $render->setElementTemplate(self::$_elementTemplate);
    }
```

詳しくは[HTML\_QuickForm\_Renderer\_Tableless](http://pear.php.net/package/HTML_QuickForm_Renderer_Tableless/docs/latest/HTML_QuickForm_Renderer_Tableless/HTML_QuickForm_Renderer_Tableless.html)をご覧ください。

アダプターがRENDERER\_SMARTY\_ARRAYレンダラ(0)の場合は

[HTML\_QuickForm\_Renderer\_ArraySmarty](http://pear.php.net/manual/ja/package.html.html-quickform.html-quickform-renderer-arraysmarty.php)クラスのオブジェクトが渡され[API](http://pear.php.net/package/HTML_QuickForm/docs/latest/HTML_QuickForm/HTML_QuickForm_Renderer_ArraySmarty.html)の操作([setErrorTemplate()](http://pear.php.net/package/HTML_QuickForm/docs/latest/HTML_QuickForm/HTML_QuickForm_Renderer_ArraySmarty.html#methodsetErrorTemplate)でエラーテンプレートが指定できたり、[setRequiredTemplate()](http://pear.php.net/package/HTML_QuickForm/docs/latest/HTML_QuickForm/HTML_QuickForm_Renderer_ArraySmarty.html#methodsetRequiredTemplate)で必須項目のときのテンプレートが指定できたりします。

## 外部リンク ##

  * [HTML\_QuickFormドキュメント](http://pear.php.net/manual/ja/package.html.html-quickform.php)
  * [PEAR HTML\_QuickForm入門ガイド](http://www.planewave.org/translations/quickform/html_quickform.html)
  * [(英語)実例が豊富なチュートリアル](http://www.sklar.com/talks/index.php/nyphp-quickform/0)
  * [(英語)Smarty meets QuickForm](http://davidmintz.org/presentations/show.php/QuickForm_and_Smarty/12)