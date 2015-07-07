# 導入 #

BEARはクラスオートローダーが標準になっているので、PEARやZendのクラスを利用するときにrequireやincludeなどを省略することができます。

# 詳細 #

読み込んでいないクラスをnewしたりプロパティにアクセスしようとするとクラスファイルが自動で呼ばれます。アプリケーションのクラスはApp/以下に配置します。例えば認証クラスだと`App/Auth.php`で`App_Auth`というクラスを利用します。

例　`App_Daemon_Chat`クラス


  * path: `<プロジェクトルート>`/App/Daemon/Chat.php
  * class: `App_Daemon_Chat`

## オートローダーのオン、オフ ##

デフォルトではオンになっているオートローダーをオフにするには以下のようにします。

```
spl_autoload_unregister(array('BEAR', 'onAutoLoad'));
```

再びオンにするには
```
spl_autoload_register(array('BEAR', 'onAutoload'));
```

とします。

## 注意 ##

unserialize()で復元したオブジェクトのクラス定義がされてない場合, class\_exists()で第２引数にfalseを指定せずにクラスが存在しない場合 でもオートローダーが呼ばれます。