# 導入 #

ページにはclickという仕組みがあり、ページのイベントを指定してページを実行することができます。リンクによるクリックと、リンククリックによる実際の遷移の他に、リダイレクト時に仮想的な「クリック」でページ遷移することもできます。

クリックはページのページの”モード”を規定するものともいえます。クリックに応じてリストの順番を並び替えたりフォームの確認画面など出したりできます。

# 動作 #
## {a}タグ ##

テンプレートにBEARの{a}タグで記述します。{a}タグはHTML標準の`<a>`タグと上位互換性があります。拡張機能が２つあり１つはクリックで１つは非スカラー値の変数渡しです
```
//printイベントを発生
{a click=print}印刷ページ{/a}
```

```
//$valuesを渡して/にリンク
{a href="/" val=$values}ルート{/a}
```

```
//HTML属性と共存
{a click="print" val=$values target="_new"}新しいページで印刷用ページを開く{/a}
```

htmlの`<a>`タグと違ってhrefを省略できます。省略した場合は自身のページをさします。

### click属性 ###

click で指定されたイベントはページのonClickメソッドで実行されます。onClickメソッドはonInit()の前に実行されます。onClickメソッドでreturn falseすると処理はそのメソッドで終了します。以下はリストの並びをクリックで変えるためのUpクリックとDownクリックを作成した例です。

> ページ

```
public function onClickUp(){
  $this->_order = 'DESC';
}
public function onClickDown(){
  $this->_order = 'ASC';
} 
```

> テンプレート

```
{a click=up}昇順{/a} | {a click=down}降順{/a}
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
$options['click'] = 'up';
$options['val'] = $user;
BEAR_Page::redirect('/usr/entry', $options);
```