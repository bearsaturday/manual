# 導入 #

リソースからリンクを辿って関連リソースを取得することができます。htmlで`<a>`タグに相当する機能です。Webでユーザーはリンクをたどるだけで違うページにアクセスできます。ページの取得方法はカプセル化され、ユーザーはページの取得法を学習する必要がありません。

BEARのリソースリンクも同じです。リソースの関係はリソース内でカプセル化され、クライアントからはリンクを参照するだけで関連リソースに繋がります。


# 詳細 #

## コード例 ##

### ページ(呼び出し側) ###
ユーザーID1のユーザーの最新記事、およびコメントを取得する例です。
ユーザー情報からブログ、最新記事、コメントとリンクされていて、それらリソースを全て取得して一度に全てテンプレートにアサインする例です。

```
$id = 1;
$resource = BEAR::dependency('BEAR_Resource');
$params = array(
  'uri'=>'user/login',
  'values'=>array('id' => $id),
);
$resource->read($params)->link('blog')->link('latest_entry')->link('comment')->set('user');
```

全てのリソースが一度にテンプレートにアサインされます。
リンクしたリソースの全ての値はgetBody(true)で取得できます。
```
$resource->read($params)->link('blog')->link('latest_entry')->link('comment')->getBody(true);
```

以下のような場合は最初のログインの値だけが取得できます。
```
$resource->read($params)->link('blog')->link('latest_entry')->link('comment')->getBody();
```
### リソース ###

onLinkメソッドを用意して、リンク名をキーに、URIを値にしたリンク情報連想配列を返します。

リンクには２種類の方法で指定できます。シリアライズされたURIでクエリーを指定する方法とCRUDオペレーションの時のようにパラメーターで指定する方法です。

例）クエリー付きURIを指定する方法

```
/**
 * リンク 
 * 
 * @return array リンク
 */
public function onLink($values)
{
    $links = array('blog'    => 'user/blog?id=' . $values['blog_id'], 'friends' => 'User/Friends?user_id=' . $values['id']);
    return $links;
}
```

例）パラメーター連想配列で指定する方法

```
/**
 * リンク 
 * 
 * @return array リンク
 */
public function onLink($values)
{
    $blog = array('uri' => 'User/Blog', 'values'=>array('blog_id'=>$values['blog_id']);
    $firends = array('uri' => 'User/Friend', 'values'=>array('user_id' => $values['id']));
    $links = array('blog'    => $blog,  'friends' => $friend);
    return $links;
}
```

## デバック ##
開発時に上記のように内容を確認するには最後にp()をつけます。

```
$resource->read($params)->link('blog')->link('latest_entry')->link('comment')->set()->p();
```

このように画面に表示されます。

![http://www2.bear-project.net/image/links.png](http://www2.bear-project.net/image/links.png)