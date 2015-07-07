# 導入 #

BEARはリソース指向のフレームワークです。 モデルをオブジェクト設計されたものとして扱わず、リソースとして扱います。BEARのリソースもHTMLと同じようにROA(リソース指向設計）の次の４つの特徴を持ちます。

  * アドレス可能性
  * 統一インターフェイス
  * ステートレス
  * リンク

リソースアクセスの結果はリソースに関わらず同じRo型（リソースオブジェクト型）が返ってきます。リソース内部でエラーや例外が発生したりしてもそれをアクセスしたクライアントには同じRo型が返ってきます。

HTMLと同じような仕組みと考えていただけると分かりやすいと思います。HTMLにはURLがあり、操作は制限され(GET/POST)リソースリクエストに状態を持たず他のリソースにリンクがが用意されている場合があります。一方、BEARもリソースにURIがあり、操作は制限され(CRUDやlink)、リソースリクエストに状態を持たず他のリソースにリンクが用意されている場合があります。


# 詳細 #

リソースが何処にどのようなフォーマットで保存してあるかに関わらず利用する側は\*共通の方法\*でアクセスを行います。
リソースアクセス結果もリソースオブジェクト(RO)として\*共通のフォーマット\*で受け取ります。

データがファイルにテキスト/CSVファイルでもあってもDBレコードでもあってもリモートサイトのXMLでもあっても、あるいはリソースの状態（中身）がPEARエラーや例外でさえ、受け取るのは同じリソースオブジェクト(Ro)です。

ROA４つの特徴を以下に順番の説明します。


## アドレス可能性（URI） ##

リソースはURIを１つだけ持ち、URIを特定するとリソースが特定されます。リソースはアプリケーションリソースとアプリケーション外部の外部リソースがあり、アプリケーションリソースはプロジェクトフォルダの<プロジェクト>/App/Ro/（リソースオブジェクトフォルダ）に設置します。


### アプリケーションリソース ###

アプリケーションリソースのうち、拡張子があるファイルをスタティックリソースとして扱います。csvファイルやtxtファイル、ymlファイルなどは内容が読み込まれパースされ配列としてクライアントに返されます。スタティクリソースは原則時間無制限でキャッシュされます。

BEARのURIは基本的にURLと同じフォーマットですが、アプリケーション内部がもつアプリケーションリソースはスキーマやサービス（ホスト名）を省略します。※scpコマンドでローカルファイルを指定するときに`<localhost>`は省略できるイメージです。

#### スタティックリソース ####

例）サービス規約リソース

| URI | Service/regulation.txt |
|:----|:-----------------------|
| リソースファイル | App/Ro/Service/regulation.txt |

※read操作をすると単一の文字列がbodyに入ったリソースオブジェクトがクライアントに返ります。

例）サービスQ&Aリソース

| URI | Service/qa.csv |
|:----|:---------------|
| リソースファイル | App/Ro/Service/qa.csv |

※read操作をするとQ&Aが配列になったbodyのリソースオブジェクトがクライアントに返ります。※リソースアクセスのオプションでページグネーションが利用できます。

スタティックリソースや、検索性や更新性などを考慮してもスタティックファイルとして管理した方が向いている用途用のリソースです。リソースはROリソースなので結果はキャッシュされません。

#### fileスキーマ ####

| URI | [file://path/to/data.yml](file://path/to/data.yml) |
|:----|:---------------------------------------------------|

fileスキーマでもスタティックファイルを利用できます。フルパスでファイルを指定します。値は常にキャッシュされます。

#### Roリソース ####

BEARが標準的にあつかうアプリケーションダイナミックリソースです。引数やDBや他リソースなど外部要因に依存してレスポンスが変わります。内部でロジックを実装できるためです。BEAR\_Roを継承した個別のリソースクラスを用意し、onRead()やonLink()などリソース操作に応じたメソッドを用意します。

例）ユーザーのプロフィールリソース

| URI | User/Profile |
|:----|:-------------|
| リソースファイル | App/Ro/User/Profile.php |
| クラス名 | `App_Ro_User_Profile` |

このRoリソースが一般的web dbアプリにおいて標準的なリソースです。サービスレイヤーとして機能します。内部でDBアクセス、他webサービスの呼び出しやドメインロジック実装を行います。

#### httpリソース ####

httpリソースは`http://`で始まるwebのURLを指定します。サイトがRSS(xml)のサイトなら配列データとして、プレーンHTMLなら文字列として取得されます。

#### オリジナルスキーマリソース ####
レガシーなモデル群や、他のフレームワーク、ライブラリを利用したモデルやデータソース、他リモートAPIを利用するときにオリジナルのスキーマリソースを用意することができます。

例）
例えば古い社内システムのデータアクセスにAPIやクラスファイルが用意されているとしましょう。myofficeなどというスキーマを設計してURIによるリソースアクセスできるようにできます。

例）社員番号1600の社員プロフィールリソースのURI例
```
myoffice://self/staff/profile?id=1
```

※selfはローカルホストの自分自身のサービスを表します

URIとクエリーを受け取った”myoffice”スキーマ実行クラスが社内システムにアクセスしその値を返します。

ページから直接、外部オブジェクトやAPIを利用することに対して、オリジナルスキーマを用意してURIを持つリソースとして扱う事のメリットはなんでしょうか？

- スタティックファイルを直接ページから扱わずスタティックリソースとして利用する場合と同じです。すなわち共通のキャッシュ機構、ページグネーション、ログ管理、レイジーセット、ビューからのPull取得など共通のリソースユーティリティが利用できます。またコンポーネント間の結合が疎になるので、メンテナンス性も向上するでしょう。「実装に依存」してたリソースクライアントが「リソースAPIへの依存」へと変わるからです。ログやデバッグが容易になります。

## 統一インターフェイス ##

| **HTTP** | **DB** | **BEAR** |
|:---------|:-------|:---------|
| PUT      | CREATE | create   |
| GET      | SELECT | read     |
| POST     | UPDATE | update   |
| DELETE   | DELETE | delete   |

リソースは単なるデータと違い、操作のインターフェイスが持てます。ですが自由にインターフェイスを持てるわけではなく、限られたインターフェイスしか持てません。この制限されたインターフェイスをRESTではユニファイドインターフェイスといいます。BEARのリソースは CRUD 、create(作成), read(読み込み）, update（変更), delete(削除） の操作しかできません。※

もしブログの記事リソースに「検索」の機能を加えたいときでもBlog/Entryエントリーに"検索"という操作はできません（検索インターフェイスはもてません）かわりに(Blog/Entry/Search)というリソースを作成してreadという操作を行います。

インターフェイスは４つ全てを備える必要はありません。必要なインターフェイスだけを実装します。実装されていないインターフェイス（メソッド）でアクセスされると400(Bad Request)が返されます。HTTPでは405(Method not allowed)に相当しますが、BEARでは簡略化した３つのコードしかもたないために400(Bad Request)が結果コードとして返ります。

スキーマによっても利用できるインターフェイスは変わります。例えば現在httpリソースはread操作しか用意されていません。mもしread以外の操作を行うとコード400(Bad Reqeust)のリソースオブジェクトが返されます。

## ステートレス ##
オブジェクト設計されたモデルと違いリソースは通常状態を持ちません。リクエストの準備をすべて済ませてからリクエストは１度に行います。HTTPと同様です。単純化した例を下にコードで示します。

例）オブジェクトモデル

```
$user = new User();
$user->id = 3  //id=3という状態
$user->read();
```

BEARリソースアクセス

```
$params = array('uri' => 'User?id=3');
$resource->read($params)->getBody(); //リソース内部ではid=3の情報は保持していない

または

$params = array('uri' => 'User', 'values' => array('id' => 3);
$resource->read($params)->getBody();;
```

ステートレスのリクエストは単純な分、失敗のリカバリーや記録、キャッシュ化が容易といった特徴があります。

## リンク ##

Roリソースはリンクによってリソースと他のリソースを結びつけることができます。例えばユーザーリソースは\*ブログリンク\*よってブログリソースにリンクされます。ブログリソースは\*最新記事リンク\*や\*人気記事リンク\*で記事リソースにリンクされます。

リンクはリソース内部でカプセル化され、外部からはリソースからはリンクをたどるだけです。htmlのaタグのイメージです。hrefで指定されたリンク情報はクライアントは管理しません。利用するだけです。もしリンク先リンク方法が変わったとしても利用の仕方は変わりません。

roリソース以外のリソースにもリンクできます。例えばdbのコラムにhatena\_idとあれば、それをつかってhatena web apiに直接リンクを張る事ができます。リソース間の結合をROAにしているからリンク記述も容易です。基本的にはリンク先がどんなデータソースでもURIを指定するだけです。この点がDBのリレーションやいわゆるアソシエーション機能とBEARのリンクと大きく違うところです。

## リソースクライアント ##

ページクラスでは初期化ハンドラでリソースアクセスクラス(BEAR\_Resource)をつかってリソース操作をし単数または複数のリソースをアクセスしてそのリソースをread、続けてページにsetし出力ハンドラでHTMLとして可視化します。

リソースはページ以外からも扱えます。CLIやソケット通信もクライアントになります。作成したリソースはページクラス以外からでも動作するので以下の用途にも使えます。

  * CLIでのアプリケーション初期化、メンテナンス
  * cronによるバッチ処理
  * cometの時のソケットアプリケーションサーバー

他言語やリモートサイトからはRESTやソケットを介してアクセスします。

## リソースアクセスに必要なもの ##

BEARでのリソースアクセスは以下を指定して行います。

  * メソッド（必須）
  * URI（必須）
  * 引数
  * オプション

メソッドは４つしかありません。いわゆるCRUD(create/read/update/delete)インターフェイスでHTTPのPUT/GET/POST/DELETE、DBではCREATE/SELECT/UPDATE/DELETEに相当します。全てのリソースにこの４つのアクセスでしかリソースアクセスできず、RESTではこれをユニファイド（統一された）インターフェイスといいます。

リソースはURIと引数で特定されます。例えばid=5のユーザーはURIは'User'、引数はarray('id'=>5)です。これをクエリー形式で`User?id=1`とURIだけで表現することもできます。
リソースをreadして得られる結果をRESTではリソース状態といいます。

オプションは、キャッシュのリソース状態の加工（ページグネーションやコールバック）やキャッシュ使用などの指定に利用します。ユーザー定義したオプションも利用できます。

何を（URI + 引数）どうやって（オプション）どうするのか（メソッド）と考えてみましょう。

## リソースオブジェクト ##

リソースアクセスの結果はリソースオブジェクト(Ro)として返ります。Roはミニwebのようなものです。４つのプロパティを持ちます。

  * コード
  * ヘッダー
  * ボディ
  * リンク

### Roコード ###

コードはHTTPレスポンスコードに準じた3つのコードがリソースの状態を表しています。

  * 200 OK 問題がない
  * 400 Bad Request  呼び出し方法に問題がある
  * 500 Internal Resource Error リソース内部に問題が発生した

参考: http://ja.wikipedia.org/wiki/HTTPステータスコード

### Roヘッダー ###

リソース状態（ボディ）のメタ情報が入っています。例えば、ユーザーリソースの結果が配列で返されたとします。それが「何件あるのか」「いつ作成されたのか」といったリソース状態そのものを表しているのではないが、リソース状態に依存した属性情報を格納するのに用います。HTMLでいうと`<header></header>`の部分です。HTMLの`<body>`のメタの属性情報（ドキュメントタイトル、作成日、使用文字コード）に`<header>`が使われています。

### Roボディ ###

リソース状態です。HTMLでいうと`<body></body>`の部分です。通常、ビューにアサインされる配列や文字列が入りますが、オブジェクトであってもかまわず型に制限はありません。

### Roリンク ###
HTTPと違うのはこの部分です。リンクが独立したプロパティになっています。
リソースは他のリソースにリンクを張れます。HTMLでいうと<a>タグで指定された他ページへのリンクです。リンクがあるおかげで、関連リソースへのアクセス方法を知る事なしに（カプセル化して）リソースからリソースに情報をたどることができます。HTMLでも最も重要な機能です。<br>
<br>
<br>
<h2>サンプルコード</h2>
<h3>リソースの取得とページ化</h3>

例をあげます。「明日の天気」はリソースです。リソース状態が「晴れ」です。<br>
リソースはアドレス(URI)を持ちます。<br>
<br>
リソースにアクセスするためにはリソースアクセスクラスである、BEAR_Resourceクラスを使います。bearコマンドで作成したApp_PageにはページプロパティとしてBEAR_Resourceオブジェクトが注入されプロパティオブジェクトとして利用可能になっています。<br>
<br>
<pre><code>public function onInject()<br>
{<br>
  $this-&gt;_resource = BEAR::dependency('BEAR_Resource');<br>
}<br>
</code></pre>

リソースを読むのにリソースの場所（URI)と、読み出しに必要な引数「地域は東京」など(values)が必要です。読み込みを実行した後にgetBody()でその結果の内容を取得します。<br>
<pre><code>$params = array(<br>
  'uri'=&gt;'Whether/Tommmorow',<br>
  'values'=&gt;array('region' =&gt; 'tokyo')<br>
);<br>
$wheather = $this-&gt;_resource-&gt;read($params)-&gt;getBody();<br>
</code></pre>

結果をビューにセットします。<br>
<br>
<pre><code>$this-&gt;set('wheater', $wheater);<br>
</code></pre>

以下のように「[<code>http://capsctrl.que.jp/kdmsnr/wiki/bliki/?FluentInterface</code> 流れるようなインターフェイス]」(fluent interface)でsetできます。※この場合のreadはonInit()終了時にlazy実行されます。<br>
<br>
<pre><code> $this-&gt;_resource-&gt;read($params)-&gt;set('wheater');<br>
</code></pre>

リソースの考え方はwebそのものです。例えばhttp://www.example.com/wheather /tommorow?location=tokyoというURIリクエストそのものがリソースで、表示されたページで得られる結果がリソース状態として考えると上記のリソースアクセスが理解しやすいでしょう。<br>
<br>
<h2>ダイナミックリソースとスタティックリソース</h2>

webアプリは一般にデータの格納にDBを使います。DBやリモートサイトのXML/JSON、動的にリソース状態が変化するこれらのリソースをダイナミックリソースと呼びます。<br>
<br>
対してスタティックなファイルをリソースファイルとして配置することもできます。これがスタティックリソースです。CSVファイルやYAML、iniファイルなどのスタティックなデータファイルを設置できます。<br>
<br>
例えば郵便番号CSVファイルをそのままApp/Ro/以下のフォルダに配置すると、そのcsvファイルはスタティックリソースとしてそのまま使えます。クライアントから読み込むとPHPの連想配列になっています。<br>
<br>
BEARではURIに拡張子がないものをダイナミックリソース、あるものをスタティックリソースとして扱います。ダイナミックリソースファイルは拡張子phpのphpファイルです。<br>
スタティックリソースも、キャッシュやページングといった機能はダイナミックリソース同様に扱えます。CSVファイルのデータがDBと同じようにページングできます。<br>
<br>
<br>
<br>
<h2>リソースファイルの種類</h2>

ローカルリソースファイルはCRUD,それぞれのメソッドを記述するApp_Roクラスファイルか、インターフェイスを持たないファイル関数ファイル、あるいはスタティックファイルのいずれかのファイルで用意されます。<br>
<br>
<h3>リソースオブジェクトクラス(BEAR_Ro, App_Ro)</h3>

<code>BEAR_Ro</code>または<code>App_Ro</code>クラスを継承して個別のリソースクラスを実装します。以下の用にインターフェイスを実装します。<br>
<br>
<pre><code>//ユーザーリソース<br>
class User extend App_Ro<br>
{<br>
  public function onCreate(array $values){}<br>
  public function onRead(array $values){}<br>
  public function onUpdate(array $values){}<br>
  public function onDelete(array $values){}<br>
  public function onLink(array $links){}<br>
}<br>
</code></pre>

readしかできないリソースならonReadだけとリソースに応じて実装します。実装必須のメソッドは特にありません。<br>
<br>
<h3>リソース関数ファイル</h3>

<blockquote>新規作成には非推奨ですが、レガシー関数ファイルを使いたい場合に関数もリソースファイルとして使用できます。<br>
関数のみのリソースファイルです。CRUDインターフェイスを持ちません。関数ファイルではリソースそのものに動作を持たすようなネーミングになります。addUserとかeditUser、もしくはuser?mode=add やuser?mode=editという風に引数に動作を表すようにします。</blockquote>

<h2>リソースオプション</h2>
<h3>readオプション</h3>

<blockquote>リソースをread(取得)する時に、「どのように取得するか」「取得した後の後処理をどうするか」をリソースreadオプションとして指定できます。以下の例では以下のオプションを同時に設定しています。</blockquote>

<ol><li>キャッシュキーとキャッシュ時間を指定<br>
</li><li>１ページ５アイテムにページング<br>
</li><li>各要素をすべてHTMLエスケープ関数に渡して<br>
</li><li>App/view/elements/entiries.tplというテンプレートにアサインする</li></ol>

というオプションを同時に指定しています。キャッシュオプションを指定すると、リソース取得（DBでのセレクト）だけでなく、オプション処理全てがキャッシュされます。<br>
<br>
<pre><code>$options = array();<br>
$options['cache']['life'] = 10; // キャッシュ時間<br>
$options['cache']['key'] = $id // キャッシュキー<br>
$options['pager'] = 5; // ページャー<br>
$options['callbackr'] = array('App', 'htmlEscape'); //コールバック<br>
$options['template'] = 'list/user'; // リソーステンプレート指定<br>
$params = ('uri'=&gt;$uri, 'values'=&gt;$values, 'options'=&gt;$options);<br>
$resource-&gt;read(params)-&gt;set('entry', 'object');<br>
</code></pre>

なお、readオプションで指定するページングやテンプレートアサインなどのポストプロセスは以下の順番で実行されます。<br>
<br>
<ol><li>pager (ページグネーション)<br>
</li><li>callback　（コールバック）<br>
</li><li>callbackr　（再起）<br>
</li><li>template (テンプレートにセット）</li></ol>

※callbackは指定したコールバック関数（メソッド）にcall_user_funcを使ってリソース結果配列が渡され、callbackrではarray_walk_recursiveで結果の各要素が渡されます。<br>
<br>
<h3>POE(Post Once Exactly), CSRF(クロスサイトスクリプトフォージェリ対策)オプション</h3>

RESTの用語で「サイドエフェクト」「べき等性」という用語があります。<br>
<br>
CRUD の4つのインターフェイスのうち、readはどのようなreadであってもリソース状態（内容）は変更されません。対してupdate、deleteはリソースの内容に変更があります（サイドエフェクトがあるといいます）<br>
<br>
一方、updateとdeleteは同じ操作を何回してもリソース状態の変更はありません。（「id=1の nameを"kuma"に」（updae) 「id=2のデータを消去」(delete)は同じ事を何回実行しても2回目からリソース状態に変化はありません。）<br>
<br>
<blockquote>createは違います。「name=kumaでユーザーリソースを作成」を3回実行すればリソースが3つ作成されてしまいます。「べき等性」がないといいます。フォームの多重送信の問題はこれです。また通信不良の多い携帯のwebでも問題になることがあります。</blockquote>

<blockquote>例）通信不良時のcreateの問題</blockquote>

<blockquote>クライアント: ユーザーID=kumaでユーザー登録サブミット→ サーバー：DBに登録し「登録しました」画面を表示。ところが通信不良でこのレスポンスが返らない。→  クライアント: おかしいなとユーザーID=kumaでもう一度ユーザー登録サブミット →  サーバー：そのユーザーは登録済みです画面を表示</blockquote>

<blockquote>※サーバーもクライアントも正しい動作をしても、ユーザー体験として成立しない。</blockquote>

リソースリクエストのpoeオプションでこの問題が解決できます。<br>
<br>
<pre><code>$values  = array('id'=&gt;'bear');<br>
$options = array('poe'=&gt;true);<br>
$params = ('uri'=&gt;'user', 'values'=&gt;$values, 'options'=&gt;$options);<br>
$this-&gt;_resource-&gt;create(params)-&gt;request();<br>
</code></pre>

poeオプション指定時には全く同一のリソースcreateリクエストはキャッシュされた結果を返すのみで、onCreateメソッドやリソース関数は１度しか実行されません。これによりDBのinsertアクセスは一度だけ、結果は同じものが何度でも返る、という処理が実現でき上記の通信不良の問題も起りません。DBのinsertアクセスは一度だけですが、「登録できました」画面は何度リクエストしても表示されます。<br>
<br>
csrfオプションも同様です。<br>
<pre><code>$values  = array('id'=&gt;'bear');<br>
$options = array('csrf'=&gt;true);<br>
$params = ('uri'=&gt;'user', 'values'=&gt;$values, 'options'=&gt;$options);<br>
$this-&gt;_resource-&gt;create(params)-&gt;request();<br>
</code></pre>

csrfのチェックをパスしない場合は例外が投げられます。poeの場合は同じトークンでのリクエストがあればリソースリクエストを行いません。(エラーメッセージは例外などはありません。)<br>
<br>
リクエスト単位ではなく全体に適用するにはapp.ymlで指定します。<br>
<pre><code>BEAR_Resource_Request:<br>
  poe: false<br>
  csrf: false<br>
</code></pre>

どちらもトークン用のストレージに標準でセッションを使っています。オプションを利用していてセッションが切れてしまうと、そのサブミットはcsrfチェックに引っかかってしまいます。<br>
<br>
トークン用のストレージをセッションを用いずアプリケーションが独自に用意することができます。`BEAR_Form_Token'のインジェクターonInjcet()を切り用意したサービスをインジェクトします。<br>
<br>
<a href='https://github.com/koriym/beardemo.local/blob/master/htdocs/form/poe/index.php'>beademo form/poe/index.php</a>

<h2>リソースオブジェクト(BEAR_Ro)</h2>

リソースファイルでリソース状態を返すには2つ方法があります。1つは得られたリソース結果をそのまま連想配列やスカラー値として返す方法です。DBのクエリー結果などの連想配列変数をそのまま返すことができます。もう1つの方法はリソースオブジェクト(BEAR_Ro)を利用する方法です。<br>
<br>
例）サーバー変数をかえすServerリソースにアクセス<br>
<br>
bodyだけ（配列変数）を返す場合<br>
<pre><code>/**<br>
  * @return array<br>
  */<br>
public function onRead($values)<br>
{<br>
  $array = $_SERVER;<br>
  return $array;<br>
}<br>
</code></pre>
ヘッダーにcount情報を付加したRO（リソースオブジェクト）を返す場合<br>
<pre><code>/**<br>
  * @return BEAR_Ro<br>
  */<br>
public function onRead($values)<br>
{<br>
  $result = ...<br>
  $ro = $this-&gt;setHeader('count', count($result))-&gt;setBody($result);<br>
  return $ro;<br>
}<br>
</code></pre>

上記のいずれを返しても受け取る方はROで受け取ります。<br>
<br>
<pre><code>$ro = $this-&gt;_resource-&gt;read($params)-&gt;getRo();<br>
$body = $ro-&gt;getBody(); // bodyの取得<br>
$headers = $ro-&gt;getHeaders(); // headerの取得<br>
</code></pre>

リソースオブジェクトはwebページに似ています。最も重要なのは<code>&lt;body&gt;&lt;/body&gt;</code>タグで囲まれたボディ部分ですが、そのbodyコンテンツのメタ情報が<code>&lt;header&gt;&lt;/header&gt;</code>でヘッダーとして表されます。ページには他のページへの「リンク」があります。<br>
<br>
リソースオブジェクトのもボディ、ヘッダー、リンクのプロパティがあります。それに加えリソース状態を表すコードプロパティがあります。<br>
<br>
例をあげます。「検索」リソースではDBページャーを用いて検索結果を返します。データ量は膨大なのでDBページャーを使い1ページ辺り10件の検索結果を「検索」リソース状態として返すことにしましょう。リソースオブジェクトではbodyプロパティに結果が入ります。しかしユーザーは「1ページの検索結果」以外のデータも必要でしょう。「検索結果の全体件数」「現在表示中のページ番号」などといったメタ情報です。これは「header」プロパティに格納されます。そして最後に、次ページや前ページといった現在のリソースから他のリソースへのリンクがあります。「link」プロパティが使用されます。<br>
<br>
<h2>リソースアクセスサンプルコード</h2>


このようにBEAR_Roはリソースの状態（結果）を保持する機能と、リソースアクセスのメソッドを実装(Create, Read, Update, Delete)する機能と2つ大きな機能があります。<br>
<br>
Ro（リソースオブジェクト）はHTTP通信の仕組みに似ています。ヘッダーとボディを持ち、POST/GET/PUT/DELETEに対応するインターフェイス(onCreate, onRead, onUpdate, onDelete)を持ちます。リクエストはステートレスで行われエラーが起こっても同じフォーマットでレスポンスが返ります。<br>
<br>
<br>
<h2>クエリー形式の引数指定</h2>

下記はどちらも同じreadです。<br>
<br>
<pre><code>$this-&gt;_resource-&gt;read(array('uri'=&gt;'wheather', 'values'=&gt;array('area'=&gt;'tokyo', 'time'=&gt;'tomorrow'))-&gt;set('weather_tokyo');　<br>
$this-&gt;_resource-&gt;read(array('uri'=&gt;'wheather?area=tokyo&amp;time=tomorrow'))-&gt;set('weather_tommorow');<br>
</code></pre>

<h3>CLIでのクエリー</h3>

CLIでアクセスする場合には&をうまく認識してもらうためにシングルクオートを使います<br>
<br>
<pre><code>$ bear create 'User?name=kuma&amp;age=10'<br>
</code></pre>

<h2></h2>

<h1>RESTとBEAR</h1>

<sup>REST自身は、多くの異なる技術を利用して実装されうる高水準なスタイルなのです。RESTは、リソースや統一インターフェースの考えを持っています。つまり、全てのリソースが同じメソッドによって応答するという考えなのです。しかし、RESTはどのメソッドでなければならないのかや、メソッドの数がどれくらい必要であるのかといったことには言及していません。REST スタイルを「具現化」したものの1つが、HTTP(と、URIなどの関連する標準セット)です。もしくは、少し抽象的になりますが、Webのアーキテクチャそれ自身であるとも言えます。HTTPは、RESTの統一インターフェースをHTTPの動詞(操作)からなる特別な形で「具体化」したものと言えます。</sup><br>
<sub>作者 Stefan Tilkov, 翻訳者 松本 清一 http://www.infoq.com/jp/articles/rest-introduction</sub><br>
<br>
<br>
BEARの最もコアな部分はこのRESTに関連する部分です。HTTPやwebのアーキテクチャのように振る舞うRESTです。<br>
<br>
WebブラウザがPOST/GETでWebアプリにアクセスし、webアプリ内部ではページがリソースにCRUD操作します。リソース内部ではDBにINSERT/SELECT/UPDATE/DELETE操作をしています。<br>
<br>
ユーザーの入力からDB操作までのデータフローが比較的フラットです。途中のマッピングがなくコンポーネント間でのデータがリニアに受け渡しされやすくなっています。<br>
<br>
<h1>参考リンク</h1>

<ul><li><a href='http://ja.wikipedia.org/wiki/REST'>REST(Wikipedia)</a>
</li><li><a href='http://www.infoq.com/jp/articles/rest-introduction'>infoQREST入門</a>
</li><li><a href='http://yohei-y.blogspot.com/2005/04/rest_23.html'>REST入門</a>
</li><li><a href='http://text.art-code.org/presen/restfulreading/'>REST &amp; ROA Best Practice</a></li></ul>


注）このマニュアルではリソース、リソース状態、などのRESTの用語の言葉の厳密性はあえて問わずルーズにしています。ご了承ください。