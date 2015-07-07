# 導入 #

BEAR\_Img クラスで画像を読み込み、加工し、表示または保存することができます。一つの画像を用途によってライブラリをGD（標準的）、iMagick(フィルタなど加工にすぐれる）、Cairo(ベクター系ライブラリ）と切り替えながら使う事ができます。モバイル画像用にUA判定し最適な大きさにリサイズすることもできます。

# 詳細 #

BEAR\_Img クラスで用意されている機能は、主に画像の読み込み（リモート含む）、ローカル保存、Cairoでのアウトライン付き文字、画像の合成、 GD/iMagickでの画像のリサイズなどです。ライブラリが持つ細かな操作はイメージリソースなどを直接操作します。ライブラリ切り替え時のテンポラリーファイルの生成、削除はBEARが行います。

# 操作 #

## 携帯用にリサイズ ##
携帯エージェントに合わせた大きさにします。大きさが分からなければQVGAとします。
```
public function onInit($args)
{
     //画像ライブラリ選択 
    $img = BEAR::dependency('BEAR_Img', array('adapter'=>BEAR_Img::ADAPTOR_GD));
    $file = 'http://www.bear-project.net/image/eye.png';
    $img->load($file); 
    $img->resizeMobile();

    $img->show('gif');

}
```

## 画像ライブラリの切り替え ##
ライブラリを切り替えるにはchangeInstanceメソッドを使います。GDで読み込んでCairoに切り替えた例です。 （テンポラリー画像の作成や削除をBEARが行います)

```
$img = BEAR::dependency('BEAR_Img', array('adapter'=>BEAR_Img:: ADAPTOR_GD));
$img->load('http://www.bear-project.net/images/eye.jpg');
$img = BEAR_Img::changeAdaptor(BEAR_Img:: ADAPTOR_CAIRO);
```
> cairoの関数を使うときにはパブリックプロパティを操作します。

```
cairo_translate ($img->surface, 128.0, 128.0);
```

iMagickのMagickオブジェクトはadapterプロパティです。このように操作します。solarizeImageはソラリゼーション(太陽の効果）を行うiMagickクラスのメソッドです。

```
$img->adapter->solarizeImage(0.1);
```

## 独自アダプターの実装 ##

現在のアダプターを継承したり、新規にイメージアダプタークラスを作る場合は`$config['adpotr']`に文字列を指定します。

`App_Img_Adapotr_Hoge`クラスなら`App/Img/Adapotr/Hoge.php`を用意します。新規クラスの場合は`BEAR_Img_Adaptor`クラスを継承します。既存のクラスの拡張なら既存のものをextendsします。



# リンク #
  * [GDマニュアル](http://jp2.php.net/manual/ja/book.image.php)
  * [iMagickマニュアル](http://jp2.php.net/manual/ja/book.imagick.php)
  * [Cairoマニュアル](http://www.cairographics.org/manual/)
  * [PHPからCairoを使う](http://www.phppro.jp/phptips/archives/vol39/1)