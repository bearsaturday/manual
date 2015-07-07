## リソースのdeleteインターフェイス ##

リソースがdeleteメソッドでリクエストされたときのdeleteインターフェイスメソッドを加えます。

App/Ro/Post.php
```
    /**
     * @required id
     */
     public function onDelete($values)
    {
        $where = 'id = ' . $this->_query->quote($values['id'], 'integer');
        $this->_query->delete($where);
    }
```

Note:
> メソッドのコメントのところで@required idと記述してあるのはこの変数には'id'が必要だということです。

## コマンドラインからdeleteリクエスト ##

削除のインターフェイスがPostリソースに備わりました。コマンドラインで使ってみましょう。

まずはリソースURIのみで。

```
$ bear delete Post
```

こういう結果が返ってきました。
```
code
400
header
Array
(
    [_exception] => Array
        (
            [class] => BEAR_Annotation_Exception
            [msg] => @required item[id] is missing.
            [file] => /opt/local/lib/php/BEAR/Base.php
            [line] => 138
        )

    [_info] => Array
        (
            [RO] => App_Ro_Post::onDelete
            [required] => Array
                (
                    [0] => id
                )

            [values] => Array
                (
                )

            [doc] => /**
     * @required id
     */
        )

)
body
```

idを指定しなかったため失敗したようです。

  * code=400つまり呼び出し側に原因のあるエラーということです。BEARではHTTPのコードを簡略化した200(OK), 400(リクエストに問題がある), 500(サービスで問題がある）をリソースの状態として使用しています。

  * ヘッダーには問題解決のヒントになるようにエラーに関する付帯情報が色々入っています。

  * エラーはエラーですがCode,Header,Bodyが揃ったリソースの型でクライアントに結果が戻っています。リソースリクエストに問題がない(200=OK)場合のレスポンスと同じフォーマットです。HTTPを考えてみてください。URL入力を間違えれば404というリクエストに問題があるという400番代のエラーがリクエストに問題がない通常の場合と全く同じHTTPプロトコルを利用して返っています。webブラウザはリクエストが成功したかサーバー内で問題があったかに関心を払いません。一貫した同じ動作をするだけです。


それでは今度はちゃんとidを指定してリクエストしてみましょう。

```
$ delete Post?id=1

code
200
header
n/a
body
```

コード200=OKでヘッダーも本体も空のリソース結果が返ってきました。削除成功です。

次にこのリソースインターフェイスをページから利用できるようにしましょう。

## ページからdeleteリクエスト ##
delete.php
```
<?php

require_once 'App.php';

class Page_Delete extends App_Page
{
    public function onInject()
    {
        $this->_resource = BEAR::dependency('BEAR_Resource');
        $this->_header = BEAR::dependency('BEAR_Page_Header');
        $this->injectGet('id', null);
    }

    /**
     * @required id
     */
    public function onInit(array $args)
    {
        $params = array(
            'uri' => 'Post',
            'values' => array('id' => $args['id']),
            'options' => array('template' => 'post')
        );
        $this->_resource->delete($params)->request();
    }

    public function onOutput()
    {
        $this->_header->redirect('/index.php');
    }
}

App_Main::run('Page_Delete');
?>
```