# 導入 #

ページにはclickという仕組みがあり、ページのイベントを指定してページを実行することができます。例えば記事表示ページでは記事リソースをリスト表示しているとします。”delete”というリンクをクリックすると記事表示ページの"delete”に対応したクリックハンドラがコールされます。

リンクによるクリックの他に、リダイレクト時に仮想的な「クリック」でページ遷移することもできます。

# 詳細 #

## ハンドラ ##

"Delete"というクリックイベントが発生したときのハンドラです。

```
public function onClickDelete($val)
{
 $deleteId = $val;
}
```

クリックデータが$valに代入されます。

## {a}タグ ##

SmartyテンプレートではBEARの{a}タグで記述します。{a}タグはHTML標準の`<a>`タグと上位互換性があり`<a>`タグがサポートするタグはすべて使用できます(href, target classなど）。加えて、拡張機能が２つあり１つは\*クリック\*で１つは\*非ス
カラー値の変数\*を渡す事です。


以下、サンプルです。

```
//printクリックで印刷ページを表示
{a click=print}印刷ページ{/a}
```

```
//$valuesを渡して/にリンク, $valuesはオブジェクトや配列が可能です。URLに展開されるので長さの制限はURLに準じます。

{a href="/" val=$values}ルート{/a}
```

```
//HTML属性と共存
{a click="print" val=$values target="_new"}新しいページで印刷用ページを開く{/a}
```

htmlの`<a>`タグと違ってhrefを省略できます。省略した場合は自身のページをさします。

### click属性 ###

click で指定されたイベントはページのonClickメソッドで実行されます。onClickメソッドはonInit()の前に実行されます。onClickメソッドでreturn falseすると処理はそのメソッドで終了します。

以下はリストの並びをクリックで変えるためのUpクリックとDownクリックを作成した例です。

> ページ

```
public function onClickEdit($values){
  p($values); //$values確認
  // 編集処理
}
public function onClickDown(){
  // 消去処理
} 
```

> テンプレート

```
{a click="edit"}編集{/a} | {a click="delete"}消去{/a}
```

### val属性 ###

オブジェクトや配列をクエリー経由で渡せます。ページクラスのonInit($args)に変数が代入されます。この例は$userというユーザー情報の連想配列を次のページに渡す例です。

> ページ A　-> クリック

```
$user['id'] = $id;
$user['entry'] = $entry
$this->set('user', $user);
```

> クリック -> ページ B

```
function onInit($args){
  $id = $args['id'];
  $user = $args['user'];
}
```

> テンプレート

```
{a href="/page_b/" val=$user}ページBへユーザーIDとエントリー番号を渡す{/a}
```

## クリックリダイレクト ##

イベントを指定してリダイレクトするにはオプションのclickキーを使います。{a}タグ同様valキーも使えます。
```
$options['click'] = 'edit';
$options['val'] = $userId;
$this->_header= BEAR::dependency('BEAR_Page_Header');
$this->_header->redirect('/usr/entry', $options);
```