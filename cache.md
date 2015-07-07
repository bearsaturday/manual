# 導入 #

BEAR はキャッシュを広く使用しています。YAMLなど設定ファイルキャッシュ、リソースのreadキャッシュ、ページinitキャッシュ、インジェクトキャッシュ等です。

# 詳細 #
## アダプター ##
キャッシュの実装（アダプター）どはコンストラクタのadapterで指定します。
キャッシュアダプターはファイルベースの`PEAR::Cache_Lite`、キャッシュデーモンを使うmemcached、アクセラレーターのAPCのいずれかがが使用できます。 memcachedが速度とクラスター対応が利点です。Cache\_Liteはどちらの点にも劣りますがレンタルサーバーなど memchacedのサービスが利用できない時に有効です。webサーバーが一台の時はAPCが最速（みたい）です。

## リソースキャッシュ ##

yaml ファイルやiniファイルなどの設定ファイルの読み込みは指定しなくて暗黙的にキャッシュされます。キャッシュの製造期間は無期限ですがファイルの日付をチェックしています。キャッシュキーは自動的にapp.yamlのcoreで指定するidが付加されます。ライブにファイルをシンクするタイミングでキャッシュを消したい場合にはそのIDを変えます。


## ダイナミックリソース ##

リソースをreadアクセスするときにキャッシュが使えます。
```
$resource = BEAR::dependency('BEAR_Resource');
$uri = 'Usr/Profile';
$options['cache']['life'] = 60;
$options['cache']['key'] = "profile{$id}";  //省略可
$params = array('uri'=>$uri, 'options'=>$options); 
$resource->read($params)->set();
```

## スタティックリソース ##

xmlファイルやcsvファイルなどスタティックリソースはオプションを指定してなくても期間無制限でキャッシュされます。

```
$resource = BEAR::dependency('BEAR_Resource');
$params = array('uri'=>'Usr/Profile'); 
$resource->read($params)->set('usr1');
$params = array('uri')=> 'Usr/profile.csv'); 
$resource->read($params)->set('usr2');
```

## initキャッシュ ##

onInit()でテンプレートにアサイン(set)した値やオブジェクトがキャッシュされます。オブジェクトはシリアライズされ記録されます。

```
$config = array('cache'=>array('type'=>'init', 'life'=>60));
BEAR_Main::run('Page_Index', $config);
```

lazyセットされたリソースオブジェクトは、initキャッシュでキャッシュされるのは「リソースの取得方法」だけです。実際のリソースリクエストは毎回行われることに注意しましょう。


## pageキャッシュ ##

出力HTTPヘッダーを含むページコンテンツ全体がキャッシュされます。
インジェクターはキャッシュされません。インジェクトした内容がキャッシュキーに反映されます。

```
$config = array('cache'=>array('type'=>'page', 'life'=>60));
BEAR_Main::run('Page_Index', $config);
```

## init/pageキャッシュキー ##

キャッシュキーは省略すれば自動で付加されます。
ページクラス名、$config, ページャーキー, $args等を合成してキャッシュキーを生成しています。
詳しいロジックは[BEAR\_Page::getCacheKe()](https://github.com/koriym/BEAR.Saturday/blob/master/BEAR/Page.php)で

## キャッシュの注意 ##

  * init/pageキャッシュ利用時はonInit()内の処理がされないのでonInit()で記述しているリソースアクセス以外の処理が実行されません。（フォームの作成や他APIのアクセスなど）

  * ページキャッシュをクリアするにはBEAR\_Page::clearPageCache()を使います。
```
public function onInject()
{
   if (isset($_POST['search')){
        $this->clearPageCache();
    }
}
```


### オブジェクトのキャッシュの注意 ###

キャッシュは値だけでなくオブジェクトを含むことができます。（initキャッシュなど）オブジェクトのアンシリアライズの時にオブジェクトのクラスファイルがオートロードされますが、オートロードの命名規則にしたがわないクラスはキャッシュreadのときにオートロードエラーが出てしまいます。このようなクラスは予めrequireなどでクラスファイルを読み込んでやる必要があります。Zend系ライブラリはは恐らくすべて大丈夫です。BEARのクラスは全クラス大丈夫です。PEARもほとんど大丈夫なのですが、例えばMDB2は問題があります。`MDB2_Driver_Common`クラスはMDB2/Driver/Common.phpではなく、MDB2.phpに記述されているので以下のようにファイルを明示的に読み込んでおく必要があります。場所はApp.phpの読み込みの直後などが良いでしょう



## キャッシュクラス ##

低レベルでキャッシュ扱いたときにキャッシュクラスが利用できます。以下は基本的な例です。
```
$cache = BEAR::dependency('BEAR_Cache');
$cache->setLife(60 * 60); // 1時間のキャッシュ
$result = $cache->get($key);
if ($result === false) {
  $values = something();
  $cache->set($key, $valuse);
}
```

## キャッシュクリア ##

### init/pageキャッシュのクリア ###
自身のページが利用しているリソースにアップデートをかけたときなど、時間切れではなくて明示的にinit/pageキャッシュをクリアしたい場合があります。例えばアクションメソッドでコメントを投稿してそのコメントが即座に見れるようにするためです。以下のようにします。

```
$this->_resource->create($params);
$this->clearPageCache();
```

自分自身にリダイレクトする以下のコードのときにもclearPageCacheはコールされpageキャッシュが消去され、ページコンテンツがリフレッシュされます。
```
$this->header->redirect('.');
```


## 開発中のキャッシュクリア ##

デバックモードではどのページでもクエリーに`_cc`を付けてページ表示すると全てのキャッシュがクリアされます。

またはcliで
```
$ bear clear-cache
```
が使えます。webシェルでも使えます。

## キャシュ消去 ##

App.phpでのアプリケーション開発時にモードに応じてキャッシュを毎回消します。
```
BEAR_Util::clearAllCache(false);
```