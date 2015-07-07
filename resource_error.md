# 導入 #

リソースでのエラー発生はBEARの例外で行います。またPEARエラーや外部ライブラリ（Zend/PEAR）で発生した例外をそのまま返す事もできます。

# 詳細 #

## ソフトエラーとハードエラー ##

App/Roリソースでエラーはエラーが発生した瞬間に全ての処理を中断するエラーと、エラーが発生してもクライントにエラーコードやエラー情報を返し処理を続行するエラーがあります。ここでは前者をハードエラー、後者をソフトエラーと呼びます。

例えば認証を前提とするページで「認証リソース」が正常取得できない場合には即HTTP 503画面を出したい場合があるでしょう、続いて他のリソースを取得する必要がない場合です。これはハードエラーです。

一方、商品ページが商品リソースを取得した後に、「レコメンドエンジン」の不調により「その他おすすめ商品リソース」が取得できないとしましょう。ページを即時終了するほでではないエラーです。これはソフトエラーです。

## ソフトエラーの種類 ##

基本的に下の２種類です。例外やアノテーション、専用メソッドを使ってエラーを発生させます。

  * 400 Bad Request（呼び出し方に問題がある)
  * 500 Internal Resource Error（リソース内部でエラーが発生）

## 400 Bad Request ##

### @requiredアノテーション ###

リクエストエラーのうち「引数に必須項目が無い」というエラーは`@required`アノテーションで簡単に実装できます。

```
    /**
     * リソース読み込み - Bad Requestエラーサンプル
     *
     * 下記@requiredアノテーションで$values['name']と$values['age']両方のパラメータが必須になっています。
     * ないとコード400(BEAR::CODE_BAD_REQUEST)ののエラーROオブジェクトの返されます。
     *
     * @required name
     * @required age
     *
     */
    public function onRead($values)
    {
        return "My name is {$values['name']}, {$values['age']} yesrs old.";
    }
```

### アサーション BEAR\_Ro::assert($bool, $msg) ###

BEAR\_Ro::assert($bool, $msg)メソッドで入力項目のチェックができます。コード400のROが返されるのは同じです。

```
    /**
     * {$values['num']}は存在し、０より大きい数です
     *
     * @required num
     */
    public function onRead($values)
    {
        $this->assert($values['num'] != 0, 'num must be not zero.'); //メッセージ指定あり
        $this->assert($values['num'] > 0); //メッセージ指定なし
        return "{$values['num']}は０より大きい数です";
    }
```

### 400例外 ###
その他リソース内でリクエストエラーを発生させるには以下のようにします。
```
    public function onDelete($values)
    {
        $msg = 'このリソース削除はできません';
        $info = array('hoge'=>$fuga);
        throw $this->_exception($msg, array(
                    'code' => BEAR::CODE_BAD_REQUEST,
                    'info' => $info));
    }
```

専用のリソース例外クラスを用意することができます。

例）
App/Ro/Userの例外ファイルはApp/Ro/User/Exception.phpにBEAR\_Exceptionを継承した例外クラスを作成します。
```
App_Ro_User_Exception extends BEAR_Exception
{
}
```

## 500 Internal Resource Error ##

### 500 例外 ###
```
    /**
     * リソース変更 - リソース内部エラーサンプル
     *
     * リソース内部でエラーが起きたときはコード500(=BEAR::CODE_ERROR)の例外を発生させます。
     *
     */
    public function onUpdate($values)
    {
        $msg = '現在一切の更新はできません';
        $info = array('date'=>_BEAR_DATETIME);
        throw $this->_exception($msg, array(
                    'code' => BEAR::CODE_ERROR,
                    'info' => $info));
    }
```

PEARエラーを返しても例外と同じように500のエラーが返ります。
PEAR::MD2やPEAR::HTTP\_Clientを利用して、結果が正常かPEARエラーか調べなくてもそのまま返す事ができます。
```
$result = MDB2::connect('pgsql://usr:pw@localhost/dbnam')->query('SELECT * FROM clients');
// $resultが正常な値かクエリーに失敗したPEARエラーかわかりませんがそのまま返します。
return $result;
```

Zendライブラリなど他のライブラリを利用して例外が発生しても500エラーとして返ります
```
  $result = Zend::hoge(); // ここで例外が発生してもクライアントには500エラーとして返ります。
  return $result;
```

## ハードエラー ##

HTTPのステータス画面を出力するときにはPanda\_Exception例外をthrowします。HTTP画面を出力し処理を即終了します。リソースエラーと違いHTTPコードが全て利用できます。下記は404画面を出すサンプルです。

Panda\_Exceptionはリソースに限らず何処でもthrowすることができます。

```
     /**
     * 商品リソース
     * 
     * 404 Not foundのHTTP画面を出力します。
     *
     */
    public function onCreate($values)
    {
        throw new Panda_Exception('その商品はありません', 404);
    }
```