# 導入 #

リソースにテンプレートを指定してリソースを文字列化することができます。キャッシュオプションを併用するとCPUコストの節約に効果的です。またリソーステンプレートはUA別に用意することができます。

# 詳細 #

## リソースオプションで指定 ##

リソーステンプレートオプションしてreadするとリソースのボディとリソーステンプレートが合成された文字列がリソースとして扱われます。

リソーステンプレートはリソースread時の'template'オプションで指定します。例えば'user'を指定すると'App/views/elements/user.tpl'のテンプレートが使用され、リソースボディとリソーステンプレートが合成されページテンプレートにsetされます。

この時キャッシュオプションを指定するとボディの他にテンプレート適用した文字列もキャッシュとして保存されます。

```
        $cache = array('life' => 10);
        $options = array('cache'=>$cache, 'template' => 'user');
        $params = array('uri' => 'User', 'values' =>  array('id' => $args['id']), 'options' => $options);
        $this->_resource->read($params)->set('user');

```

## リンク付きリソーステンプレート ##

リンクのあるリソースもテンプレートオプションを指定することができます。リソースが接続されたエンティティ（かたまり）としてキャッシュがつくられます。多数のリンクや何度も発行されるSQLのコストを劇的に低減します。

リンクのキャッシュはリンクされるリソースのキャッシュと別々に生成されます。リンクされるリソースのキャッシュを再利用しやすくするためです。

例えば

  1. ユーザー → フォローしてる人  → フォロワーの最近の発言リソース
  1. ユーザー → 最近の発言リソース→ 発言につけられるお気に入りリソース

と２つのリソースを取り出す場合に２回目のユーザーリソースはキャッシュされていれば再利用されます。最初のリソース以外の全てのリンクしたリソースはまとまりとしてキャッシュされます。

以下はユーザーリソースのキャッシュを10秒間、その他リンク全てのエンティティを20秒のキャッシュに指定しています。リンクのキャッシュ時間の指定は`['cache']['link']`で指定します。

```
        $cache = array('life' => 10, 'link'=>20);
        $options = array('cache'=>$cache, 'template' => 'link/blog');
        $params = array('uri' => 'User', 'values' =>  array('id' => $args['id']), 'options' => $options);
        $this->_resource->read($params)->link(array('photo', 'blog'))->link('entry')->link(array('trackback', 'comment'))->link('thumb')->set('blog', 'object');
```

### リソーステンプレート ###

リソーステンプレートでは必ず{$body}というな名前でリソースボディがアサインされます。リソースリンクがある場合にはリンク名が入ります。


body.tpl　（リソースリンクのない場合）

```
<ul>
    <li>{$body.name}</li>
    <li>{$body.age}</li>
</ul>
```


blog.tpl (リソースリンクのある場合)

$bodyのあとにリンク名が入ります。

```
<h3>ユーザー</h3>
    <ul>
        <li>{$body.user.name}</li>
        <li>{$body.user.age}</li>
    </ul>
<h3>ブログ</h3>
    <ul>
        <li>{$body.blog.name}</li>
    </ul>
<h3>記事</h3>
<ul>
{foreach item=ientry from=$body.entry}
    <li>{$ientry.title}</li>
    <ul>
        {* コメント *}
        <li>
            {foreach item=comment from=$ientry.comment}
            <li>{$comment.title}</li>
            <ul>
            {foreach item=thumb from=$comment.thumb}
                {* コメント評価 *}
                <li>{$thumb.title}</li>
            {/foreach}
            </ul>
            {/foreach}
        </li>
        {* トラックバック *}
        <li>
            {foreach item=trackback from=$ientry.trackback}
            {* コメント *}
            <li>{$trackback.title}</li>
            {/foreach}
        </li>
    </ul>
{/foreach}
</ul>
```

リソース結果がリストのリソースを接続したリソースした場合、多重ループで取り出せるのに注目してください。記事にそれぞれのコメントが入り、コメントにそれぞれのコメント評価が入るというような、リソース結果がリストでリソース接続されているリソースの表現が簡素にできます。

## UAリソーステンプレート ##

[UAスニッフィング](agent.md)を有効にしているとUAに応じてリソーステンプレートを出し分ける事ができます。例えば同じユーザーリソースを表示する場合を考えてもiPhone 用とiPad用ではユーザーリソースの表示の仕方が変わる事があるでしょう。

その場合、user.iphone.tplテンプレートとuser.ipad.tplテンプレートを用意すれば自動的に出し分けられます。ページのリソースアクセスに変更はありません。次のページテンプレートも同じ表記です。

## ページテンプレート ##

ページテンプレートは文字列化されたリソースのプレースフォルダを指定します。

ページテンプレート例 （UAによってこの$userに$user.iphone.tplもしくは$user.ipad.tplが反映されます）

```
{$user}
```