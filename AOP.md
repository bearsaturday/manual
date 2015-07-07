# 導入 #

複数のクラスを横断する処理を独立したコンポーネントとし保守性を向上させるデザインパターンがあり、これを[アスペクト指向](http://e-words.jp/w/E382A2E382B9E3839AE382AFE38388E68C87E59091E38397E383ADE382B0E383A9E3839FE383B3E382B0.html)プログラミング(AOP)といいます。BEARのリソースクラスで使用することができます。

# 詳細 #

アスペクト指向プログラミングでは専門的用語がいくつか使われます。

処理のまとまりを関心事(concern)と呼び、そのうち継承ではうまく解決できないオブジェクトを横断する処理を「横断的関心事(Crosscutting Concern)と呼びます。ロジックとして本質的である処理（Core Concern)に対する言葉で、例えばログやセキュリティ等です。対象クラスに織り込ませたい場所をジョインポイントと呼び、アドバイスと呼ばれるコードを織り込み、全体処理となります。

※AOPのより詳しく正確な説明は、[他のサイト](http://www.google.com/search?hl=ja&client=safari&rls=ja-jp&q=アスペクト指向とは)をご覧ください。ここではBEARのリソースクラスに対するAOPを扱います。

アドバイスにはいくつか種類があります

| before | 対象の実行前に実行されるアドバイス |
|:-------|:------------------|
| after  | 実行後に実行されるアドバイス    |
| around | 前後に実行（置き換えられる）アドバイス |
| retruning | 例外が投げられず正常終了する直前で実行される時に実行されるアドバイス |
| throwing |PEARエラーや例外発生時に実行されるアドバイス |


アドバイスの指定（ポイントカット）はアノテーションと呼ばれる方法で行います。以下のようにphpdocコメントにアドバイスの種類とアドバイスクラスを指定することで行います。


```
@aspect アドバイスタイプ　アドバイスクラス
```

## アノテーションによるポイントカット ##

リソースクラス
```
    /**
     * 読み込み
     *
     * @param array $values 引数
     * 
     * @return array
     * 
     * @required limit 件数
     * 
     * @aspect before App_Aspect_Auth
     */
    public function onRead($values)
    {
```
onReadが実行される前に`App_Aspect_Auth::before()`が実行されます。


beforeアドバイスクラスはBEAR\_Aspect\_Before\_Interfaceインターフェイスを実装します。afterやaroundなどほかのアドバイスも同様です。

## beforeアドバイス ##

```
class App_Aspect_Auth implements BEAR_Aspect_Before_Interface
{
    /**
     * Auth beforeアドバイス
     * 
     * @param array $values
     * 
     * @return array
     */
    public function before(array $values, BEAR_Aspect_JoinPoint $joinPoint){
        echo     "before adivce<hr>";
    }
}
```

$valuesはジョインポイントのメソッドが受け取った引数です。$joinPointでジョインポイントのリフレクションを操作することもできます。returnで何か返すとそれがonRead($values)に渡ります。

## aroundアドバイス ##

アドバイスは複数行記述することで複数指定できます。aroundアドバイスを追加した例です。
※aroundアドバイスのみ１つだけ指定可能。

```
/**
 * @aspect before App_Aspect_Auth
 * @aspect around App_Aspect_Timer
 */
```

aroundアドバイスでは元のメソッドをproceedメソッドで呼ぶ事で前後の処理を記述できます。以下はタイマーの例です。元の処理の前後にタイマースタート、ストップを処理して元のメソッドの実行時間を計ります。

```
    /**
     * Timer aroudアドバイス
     * 
     * @param array $values
     * 
     * @return array
     */
    public function around(array $values, BEAR_Aspect_JoinPoint $joinPoint){
        // 前処理
        $startTime = microtime();
        //　オリジナルのメソッドを実行
        $result = $joinPoint->proceed($values);
        //後処理
        $time = microtime() - $startTime;
        $obj = $joinPoint->getObject();
        $obj->setBody('hello');
        echo " Time=$time <hr>";
        return $result;
    }
```
## リンク ##
  * [bear-demoアスペクト指向](http://code.google.com/p/bear-demo/wiki/aop/)
  * [BEARで始めるWebアプリケーション開発 その10「アスペクト指向プログラミングしてみる」](http://d.hatena.ne.jp/stellaqua/20100323/1269328310)
  * [wiki(ja)](http://ja.wikipedia.org/wiki/%E3%82%A2%E3%82%B9%E3%83%9A%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0)

  * [wiki(en)](http://en.wikipedia.org/wiki/Aspect-oriented_programming)