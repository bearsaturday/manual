# 導入 #

BEAR用のjQueryのプラグイン,jquery.bear.jsが用意されています。AJAXリンク、AJAXフォームの作成が行えます。

# 詳細 #

リンクまたはフォームリクエスト中はローダーIDで指定されたNow Loadingなどのエレメントが表示されます。デフォルトではidがloaderのエレメントです。

## セキュリティ ##

クッキーに保存されたセッションIDを読みAJAXリクエストヘッダーに付加することで[クロスサイトリクエストフォージェリ]（CSRFまたはXSRF）を防止する機構が実装されてます。
（この機能はセッションを利用して実装されています。セッションを無効にしている場合はこの機能を無効にしてください）
app.ymlに下記の値をセットすることで無効にできます。ただし認証が必要なサイトでは有効にすることをおすすめします。
```
BEAR_Page_Ajax:
  security_check: false
```


## 準備 ##
  1. BEAR/data/htdocs/bearのシンボリックリンクまたはコピーをhtdocsに配置します。
  1. htmlで読み込みます
```
<script type="text/javascript" src="/bear/jquery.bear.min.js?{appinfo version}"></script>
```
jquery.bear.min.jsはこの3つのファイルが[google closure](http://closure-compiler.appspot.com/home)で最適化されたものです。
  * jquery.js
  * jquery.cookie.js
  * jquery.bear.js

## AJAX ##
jQueryのセレクターでセレクトされた要素が"AJAXリンク"または"AJAXフォーム"になります。

## AJAXリンク ##

以下の例はrel=ajaxというアトリビュートを持つAタグをAJAXリンクにします。
```
$("a[rel^='ajax']").p().bearAjaxLink();
```

以下の例はid=rabbitというエレメントを"img"でクリックします。PageではonClickImg()がコールされます。
```
$("#rabbit").p().bearAjaxLink({url: "ajax.php?_cn=img"});
```

AJAXリンクで画面にあるフォームの値を読めるようにするためにはformオプションを指定します。
```
$("a[rel^='ajax']").p().bearAjaxLink({form: true});
```

### AJAXリンクレスポンス ###

AJAXリクエストされたPageスクリプトでは、どのように動作するかをコマンドの集まりとして返します。

まずはインジェクターでサービスを利用可能にします。

```
    public function onInject()
    {
        parent::onInject();
        $this->_ajax = BEAR::dependency('BEAR_Page_Ajax');
        $this->injectAjaxRequest();
    }
```
injectAjaxRequest()はAJAXリクエストの送信元フォームの値などをonInit(array $args)に$argsに注入するメソッドです。

#### htmlコマンド ####
```
        $this->_ajax->addAjax('html', array('msg' => 'AJAXリクエスト成功!'), array('effect' => 'show'));
```
IDが"msg"というDIVなどのDOMエレメントにメッセージを"show"というエフェクトで挿入します。差し込むHTMLの指定は配列で複数してできます。

#### valコマンド ####
フォームに値を入力するコマンドでです。
```
$this->_ajax->addAjax('val', array('name' => 'チャーリー・チャップリン',
            'gender' => 'male',
            'blood' => 'O',
            'comment' => '人生はクローズアップで見れば悲劇。ロングショットで見れば喜劇。'));
```
#### jsコマンド ####
クライアントのJSのメソッドを実行します。メソッドは$.app.[メソッド]として$.appの下に設置します。
この例では$.app.demoというメソッドを{demo : "#bearlogo"}という引数でコールしています。
```
$this->_ajax->addAjax('js', array('demo' => '#bearlogo'));
```

コマンドは複数指定できます。addAjaxした順に実行されます。addAjaxで登録したコマンドの発行は以下のように行います。

```
    /**
     * 出力
     *
     */
    public function onOutput()
    {
        $this->output('ajax');
    }
```
これでコマンドがJSONに変換され、それを受けたbear.jsがJSでコマンドを実行します。


## AJAXフォーム ##
Quckformのバリデーションとエラー表示をAJAXで行います。

この例ではrel=ajaxというアトリビュートがついているformタグを持ったフォームをAJAX化します。
```
$("form[rel^='ajax']").bearAjaxForm();
```

QuickFormをつくるときに以下のように指定します。
```
    /**
     * インジェクト
     *
     */
    public function onInject()
    {
        $this->_form = array('formName' => 'form', 'attributes' => array('rel'=>'ajax'));
    }
```

送信ボタンはサブミット動作をする代わりにフォームのバリデーションを行うAJAXリクエストに変わります。該当フォーム内
でrel=formというアトリビュートを持つaタグもサブミットボタンと同じ役割をします。

エラーではエラー用のCSSが該当フォームエレメントに割り当てら、エラー内容はホバーTIPSで表示されます。

## AJAXリクエスト ##

AJAXリンクやAJAXフォームは特定のエレメントにトリガーイベント（デフォルトではクリック）を割り当ててAJAXを利用可能にするというものでしたが、JSで直接AJAXリンクを実行することもできます。

```
$.bear.ajax(options)
```

optionsでオプションを指定しますが必須なのはURLです。

## オプション ##

用意されているオプションは以下の通りです。

| オプションキー | 意味 | デフォルト|
|:--------|:---|:-----|
| url     | リクエストURL | n/a  |
| form    | ページのフォームデータを送信するか | false|
| event   | バインドするイベント | 'click' |
| loading | ローダーID | 'loader' |
| beforeSend | リクエスト前に実行するメソッド | false |
| success | リクエスト成功時に実行するメソッド |false |
| error   | リクエストエラー時に実行するメソッド | false |

## デバック用関数 ##
デバック用に変数を表示する関数が用意されています。
### サーバーサイド (PHP) ###

```
p($var);
```
ページ内でp($var);としてもAJAXリクエストではfirePHPでJavaScriptに変数表示がされます。Firefox + FireBug + FirePHPの環境で利用できます。通常のPHPリクエストと同じようにかけ、AJAX化されたらfirebug表示になります。

### クライアントサイド (JS) ###
```
$.bear.log(var);
```

FireFox + FireBugの環境でコンソールに表示されます。他ブラウザはまだ対応していません。