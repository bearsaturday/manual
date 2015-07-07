# 導入 #

phpdoc形式のコメントで、リソースメソッドのメタ情報を記述することができます。現在サポートしているのはAOPプログラミングの@aspect、必須項目を記述する@requiredの2つです。

# 詳細 #

## @aspect アスペクトアノテーション ##
@aspectで指定します。詳しくは[アスペクト指向](aop.md)をご覧ください。
リソースクラスのonCreate, onRead, onUpdate, onDeleteメソッドでのみ指定できます。

## @required 必須項目アノテーション ##
連想配列で引数を1つだけ受け取る場合、必須なキーを指定します。
例）
```
    /**
     * リソース読み込み - Bad Requestエラーサンプル
     *
     * 下記@requiredアノテーションで$values['name']と$values['age']両方のパラメータが必須になっています。
     * ないとコード400(BEAR::CODE_BAD_REQUEST)ののエラーROオブジェクトの返されます。
     *
     * @required name
     * @required age
     *
     */
    public function onRead($values)
    {
        return "My name is {$values['name']}, {$values['age']} yesrs old.";
    }
```
この例ではcreate操作のためには$values['name']と$values['email']が必要です。この条件を満たさないとBEAR::CODE\_BAD\_REQUEST(400)コードのリソースオブジェクト(Ro)が返されます。

## @optional オプションアノテーション ##

現在未実装ですが、将来の互換性のためにつける事を勧めます。オプションで指定できる引数のキーを指定します。@requiredと違い必須項目ではなくオプションで指定できるキーです。