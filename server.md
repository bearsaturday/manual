# 導入 #

ソケットサーバーを作成することができます。
複数クライアントから同時にリクエストを受け付ける'フォーク'と、順に処理する'シーケンシャル'の２つのタイプのサーバーがあります。

サーバーを作成するためには、リクエストを処理するイベントハンドラが必要です。デフォルトで用意されているBEARのリソースハンドラでは、外部RSSサイトリソース、スタティックリソースを含むBEARで扱える全てのリソースがソケット通信で取得できます。

# 用途 #

  * commet使用アプリ（リアルタイム性のあるチャットなど）
  * マッシュアップ用リモートリソース取得
  * 非BEARシステムからのBEARリソースの取得
  * ブロードキャスト機能（シーケンシャルのみ）

# サーバー #

  1. `<PEARディレクトリ>/BEAR/data/cli/server.php`のスクリプトを`<アプリケーション>/cli/`にコピーします
  1. スクリプトを編集。ポート番号とサーバータイプ、ハンドラを指定します。
```
// BEARサーバースタート
$server = BEAR::dependency('BEAR_Resource_Server');
$port = 103754;
$isFork = false;
$handlerName = 'BEAR_Resource_Server_Handler';
$server->start($port, $isFork, $handlerName);
```
  1. 起動します。
```
php <アプリケーション>/cli/server.php
```

# アクセス #

立ち上げたサーバーをテストするにはtelnetで接続します。
```
telnet localhost 103754
```

コマンドを入力します。helpは/helpで見れます。
```
<メソッド> <クエリー付きURI>
```
とします。引数はクエリー形式で与えます。連想配列やオブジェクトも可能です。ここを参考にしてください[http\_build\_query ](http://jp2.php.net/http_build_query)

これはaで受け取った文字を２度繰り返して返し、bで受け取った文字を他の接続クライアントにブロードキャストするリソース\*kodama\*をreadした例です。
```
read kodama?a=hello&b=moshi
```

このような結果が帰ります。
```
200
Content-Type: text/php

s:11:"hello hello";
```
フォーマットは上の行から、
  1. 結果コード
  1. ヘッダー情報（複数行)
  1. 空行
  1. 結果（ボディ）
と３つの情報が返されます。

このサーバーに接続していた他のクライアントにはこのような結果が返ります。
```
200
Content-Type: text/php
X-Send-Type: broadcast

s:11:"moshi moshi";
```

コードはBEAR\_Voのコードと同じくHTTPに対応していて、以下の３種類のうちいずれかです。

  * 200 OK
  * 400 呼び出し不良　(Bad Request)
  * 500 呼び出された側のエラー (Server Error)

# リソース #

サンプルのkodamaリソースです。

```
class Kodama extends BEAR_Ro
{
    /**
     * 読み込み
     * 
     * @param array $values
     * 
     * @return array
     */
    public function onRead($values)
    {
        // ブロードキャスト
        if (isset($values['b'])){
            $this->setHeader('broadcast', "{$values['b']} {$values['b']}");
        }
        return "{$values['a']} {$values['a']}";
    }
}
```
デフォルトのハンドラではbroadcastヘッダーの内容がブロードキャストされ、return値がクライアントに渡されます。

# クライアント #

非BEARシステムからBEARサーバーにリクエストする例です。

BEAR/Server/Client.phpファイルと[PEAR::Net\_Socket](http://pear.php.net/manual/ja/package.networking.net-socket.php)が必要です。

```
$client = new BEAR_Resource_Server_Client('localhost',  '10082');
$result = $client->send('read', 'kodama?a=Hello&b=Moshi');
```
結果は連想配列で返されます。