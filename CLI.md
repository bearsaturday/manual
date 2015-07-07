# 導入 #

リソースはwebを通さずにコンソールから直接アクセスする事ができます。バッチ処理にも向いています。
開発のテストにも役に立ちます。コンソールの他にデバック画面のWeb Shellでもbearコマンドが使えます。Web Shellではprinta表示でリソースの内容が表示されます。

# 詳細 #

ユーザーリソース読み込み
```
$ bear read User
```
外部リソースと内部リソース
```
$ bear read http://rss.asahi.com/f/asahi_newsheadlines
$ bear read /user/local/app1/members.csv
```
id=1で年齢が20のユーザーリソース
```
$ bear read 'User?id=1&age=20'
```

アプリケーションパスの指定
```
$ bear read 'User?id=1&age=20' --app path/to/app
```
  * --appまたは-aでパスを指定します。
  * 必ずコマンドの最後に指定する必要があります。

クエリーの他にファイルで入力を指定できます。~/user1.ymlの内容でUserを作成する場合は以下のようになります。
```
$ bear create --file=~/user1.yml User 
```

ログ集計バッチ
```
 bear update Admin/Log?logid=newuser
```

## その他 ##
App Homeの確認
```
 bear show-app
```

App Homeの変更
```
 bear set-app /home/user/app/bear.app
```

## Help ##
brear read --helpオプションで表示されるものは以下の通りです。create/update/deleteはfileオプションのみを持ちます。

```
bear read --help
```
```
Usage:
  bear [options] read [options] <uri>

Options:
  -g file, --file=file        load arguments file.
  -l length, --len=length     filter specific lenght each data.
  -f format, --format=format  var_dump<default> | table | printa | php |
                              json
  -h, --help                  show this help message and exit

Arguments:
  uri  resource URI
```

## バッチ等の時は Create ?　Update ? ##

> RESTインターフェイスで一般的なバッチ処理（掃除、集計）はどのメソッドが適切でしょうか？RESTの世界もPUTかPOSTかどちらが適切かという議論があります。

[続・コマンド的な処理をどうやってRESTfulに実装するか(岩本隆史の日記帳)](http://d.hatena.ne.jp/IwamotoTakashi/20090326/p1)

BEARでもたとえばバッチ処理をどちらの置いても変わりません。一つの指針として、そのコマンドやバッチが何度更新しても同じ（べき等性がある）= update 、べき等性が無い=createと区別してはどうでしょうか。