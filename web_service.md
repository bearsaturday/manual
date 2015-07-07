# 導入 #

BEARリソースをWebサービス(REST)としてHTTP出力できます。XML/JSON/PHPシリアライズ変数等の出力フォーマットが選べます。

# サンプルコード #

APIのインターフェイスになるページPHPのサンプルです。

```
require 'App.php';

$values = BEAR::dependency('App_Query_Validater')->validate($_GET);
$uri = BEAR::dependency('App_Router')->route($_SERVER['REQUEST_URI']);
$resource = BEAR::dependency('BEAR_Resource');
$params = array('uri'=>$uri, 'values'=>$values);
  switch ($POST['_method']) {
    case 'POST':
      $ro = $resource->create($params)->getRo();
      break;
    case 'GET':
      $ro = $resource->read($params)->getRo();
      break;
    case 'PUT':
      $ro = $resource->update($params)->getRo();
      break;
    case 'DELETE':
      $ro = $resource->delete($params)->getRo();
      break;
    default:
      $vo = new BEAR_Ro();
      $vo->setCode(BEAR_Vo::CODE_BAD_REQUEST);
      $vo->outputHttp();
    }
$ro->setBody(jsone_encode($ro->getBody()));
$ro->outputHttp();
```

mod\_rewrite使用、PUT/DELETEを許可で本格的なRESTインターフェイスになると思います。

# 結果コード #

リソースファイル内でエラーを出力すると、HTTPエラーコードとなって出力されます。

```
/**
 * @requied id
 */
 public function onRead($values){
   //phpdoc内のアノテーションにより idがなければHTTPレスポンスコードは400(Bad Request)
   ...
   $result = $db->query($sql);
   return $result;                     // PEARエラーならHTTPレスポンスコードは500(Internal Error)
   ...
   error('任意エラー');                // 500エラーを出力
```

  * MDB2エラーなどのPEARエラーは発生しただけではスクリプトは続いて実行されます。500にするためにはPEARエラーを返す必要があります。即実行停止では困る場合があるためです。