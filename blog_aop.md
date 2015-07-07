## モチベーション ##

現在のblogアプリケーションでは、記事を追加したり編集したりするときに createdAt や updatedAt をリソース内で定義して更新しています。これらの処理がより複雑でまとまったものならどうしますか？コードを集中させるために処理をまとめたメソッドを用意してそれをコールしますか？あるいはリソースの前後に行うべきトランザクションにはどう対応しましょうか。メソッド内でbegin, commitのコードをcreatインターフェイス内で繰り返し書かなくてはいけないでしょうか？

BEARではこれらの要求をリソースにまたがる「横断的」な処理としてメソッドコメント内のアノテーションで指定することができます。リソースインターフェイス内では「本質的」な処理だけをかくので可読性が高まります。

Note:
> 横断的関心事と本質的関心事に処理の関心を分離するこれらのプログラミングパラダイムをアスペクト指向プログラミング(AOP)と言います。また横断的な処理を"アドバイス"といいます。

## 日付更新 Beforeアドバイス ##
```
class App_Aspect_Created implements BEAR_Aspect_Before_Interface
    public function before(array $values, BEAR_Aspect_JoinPoint $joinPoint)
    {
        $values['created'] = _BEAR_DATETIME_;
        return $values;
    }
```

```
class App_Aspect_Updated implements BEAR_Aspect_Before_Interface
    public function before(array $values, BEAR_Aspect_JoinPoint $joinPoint)
    {
        $values['updated'] = _BEAR_DATETIME_;
        return $values;
    }
```

```
    /**
     * @aspect before App_Aspect_Created
     */
    public function onCreate($values)
    {
        $result = $this->_query->insert($values);
    }
```

## トランザクション ##

トランザクションのためのアドバイスクラス指定
App/Aspect/Transaction.php
```
    /**
     * @aspect around App_Aspect_Transaction
     */
    public function onCreate($values)
    {
        $result = $this->_query->insert($values);
    }
```

トランザクション・Aroundアドバイス
App/Aspect/Transaction.php
```
class App_Aspect_Transaction implements BEAR_Aspect_Around_Interface
{
    public function around(array $values, BEAR_Aspect_JoinPoint $joinPoint)
    {
        // pre process
        $db = $joinPoint->getObject()->getDb();
        $db->beginTransaction();
        //　proceed original method
        $result = $joinPoint->proceed($values);
        // post process
        if (!MDB2::isError($result)) {
            $db->commit();
        } else {
            $db->rollback();
        }
        return $result;
    }
}
```
## 他のフレームワークでは ##

  * CakePHPでは命名規則によって日時が自動更新される機能があります。

  * Symfony2では[Symfony2 ライフサイクル・コールバック](http://docs.symfony.gr.jp/sf2-blog-tutorial/customize/04-doctrine-timestampable.html)という手法があります。