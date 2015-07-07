# 導入 #

オリジナルのリソーススキームを用意して、特定スキームのリソースアクセスリソースのハンドラ（リソースエクスキューター）を実装することができます。特定のデータやAPIをオブジェクトを用いて利用するのではなく、リソースとして利用できます。リソース化するとページングやキャッシュなどリソースオプション、デバック機構、ページ以外からの利用（CLIやソケットサービスなど）が利用できます。

# 詳細 #

## 例 ##

SSHで接続した先のファイルの操作を行えるsshというスキームのリソースエクスキューターを作る例

uri例）

ssh://self/www.example.com/data/text.txt


というURIを用いるリソースエクスキューターを例にします。

App/Resource/Execute/フォルダにエキスキューターを設置します。ssh://で始まるuriのエクスキューターはApp/Resource/Excute/Ssh.phpになります。

このファイルでリソースURIをパースし、渡されたvaluesとoptionsでリソースの操作を行い結果を返します。
以下がコード例です。

```
class App_Resource_Execute_Ssh extends BEAR_Resource_Execute_Adapter
{
    /**
     * リソースアクセス
     *
     * <pre>
     * リソースを使用します。
     *
     *
     * @param void
     *
     * @return mixed
     */
    public function request()
    {
        // sshでリソースをアクセスするコードを記述。$this->_configにリクエストがはいっている
       $result = ...; 
       // エラーの場合
         if (!$result) {
               $config = array('info' => $info, 'code' => 500);
               throw new BEAR_Resource_Exception('SSH Error', $config);
            }
        }
        return $result;
    }
}
```

## 別パッケージに ##

アプリケーションを跨いで用いたいオリジナルのURLスキームもあるでしょう。その場合の例を示します。

例えばHogeパッケージにssh://というリソースアクセスを可能にするリソースエクスキューターを実装を考えます。

  1. App/Resource/Execute/Ssh.phpに空のクラスを設置。

```
class App_Resource_Execute_Ssh extends Hoge_Resource_Execute_Ssh
{
}
```

  1. Hoge/Resource/Execute/Ssh.phpにssh://で始まるリソースの実際のアクセスを実装します。