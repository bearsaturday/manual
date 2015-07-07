# 導入 #

インジェクションハンドラで用意（注入）された変数やサービス（オブジェクト）を使ってリソースアクセスをしページにセットします。

# 詳細 #

初期化(init)ハンドラではリソースアクセスが主な仕事です。onInject()でインジェクトした$argsに基づいてリソースアクセスします。$argsはFirePHP環境では常にFirePHPコンソールに表示されます。またリソースのセットはテンプレートではなく、まずページに対して行われます。それがキャッシュされるのがinitキャッシュです。ページの実行時(BEAR\_Main::run()のオプションで指定することができます。

onInit()内で出力をすることはできません。たとえecho文などを記述してもライブモードでは出力はすべてキャンセルされます。デバック出力や不用意なincludeによる改行の出力も同様です。※デバックモードの時はonInit()で画面出力されたものは全て破線の内側に表示され、onInit()内での出力がされた事を表します。
## 注意 ##

  * `$_GET`や`$_COOKIE`の値など外部環境に依存する値を直接取得しないようにします。（それらの依存はonInject内で$argsに対してインジェクトします。）onInit()内はあくまでインジェクトで与えられた変数やサービスだけで処理を記述します。※ページ以外のクライアントやテストのためです。

  * HTMLの組み立ては行わないようにします。


# ソースサンプル #
```
/**
 * 初期化
 *
 * @param aray $args
 * 
 * @return void
 */
public function onInit(array $args)
{
    // お知らせリソース取得
    $params = array('uri' => 'Notice');
    $this->_resource->read($params)->set();
    // idで指定されたユーザーリソースをキャッシュ取得
    $values = array('user_id' => $args['user_id']);
    $params = array('uri' => 'message', 'values' => $values);
    $options['cache']['life'] = 60;
    $this->_resource->read($params)->set();
    // 自分のユーザーリソース、ブログリソース、最新記事リソース、コメントリソースをリンクを使って取得
    $params = array('uri' => 'user', 'values' => $values, 'options' => $options);
    $this->_resource->read($params)->link('blog')->link('latestentry')->link('comment')->getBody(true)->set();
    // 時間をセット
    $this->set('time', time());
}
```