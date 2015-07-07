HelloWorldをsiegeでベンチマーク。APC使用してライブモードのBEARとプリント分だけの素のPHPで比較。
括弧内はAPC不使用。CoreDuo 1.6G Mac Mini OSX10.5 (付属apache2/php5.2) で測定

BEAR　(APC)

```
Transactions:                6020 hits (1054 hits)
Response time:                0.02 secs (0.14 secs)
Transaction rate:          198.68 trans/sec (35.58 trans/sec)
```
PHP
```
Transactions:               16389 hits (14033 hits)
Response time:                0.01 secs (0.01 secs)
Transaction rate:          546.66 trans/sec (466.21 trans/sec)
```
## 結果概要 ##

トランザクション比で素のPHPの 36%(7.63%)のパフォーマンス。APC使用では5.58倍のパフォーマンス（※一般的なDBアプリの場合は２倍程度に落ちると思われる）

  * BEARページスクリプト
```
<?php
require_once 'App.php';
class Hello_World extends BEAR_Page
{
    public function onInit($args){
        $this->set('title', 'Test');
        $this->set('message', 'Hello World');
    }
    public function onOutput()
    {
        $this->display('/hello/world.tpl');
    }
}
new BEAR_Main("Hello_World");
```
  * BEARページビュー
```
<html>
<head>
<title>{$title}</title>
</head>
<body>
{$message}
</body>
</html>
```
主要フレームワークとの比較
編集部分

（ソースは Simple is Hard）
※素のPHPを100%とした場合のパフォーマンス.
| PHP | 100% |
|:----|:-----|
| CodeIgniter | 50.0% |
| BEAR(APC)| 36%  |
| Zend Framework 1.6.0-rc1 | 21.2% |
| Symfony | 16.3%  |
| BEAR | 7.63% |
| CakePHP | 4.2% |

※最低限のフレームワークの動作テスト。これをもって全体的なパフォーマンスの比較にはならないのには注意。
※BEARはAPC、及びアクセラレーター無しで動作、他のフレームワークは不明。（多分APC?)
参考リンク
編集部分

  * [Simple is Hard](http://talks.php.net/show/froscon08/24) （各フレームワークのパフォーマンステスト）