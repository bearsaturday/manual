# 導入 #
twitterタイプのサイトのサンプルトップページです。
ログイン・ログアウト、エントリー表示、投稿、消去ができます。

# 詳細 #

このページでは以下の機能が実現されています。

  * ログイン状態による画面変化
  * ログインフォーム
  * エントリー投稿フォーム
  * エントリー投稿アクション
  * initキャッシュ
  * エントリーの一覧表示
  * エントリーのページグネーション
  * テストのためのダミーユーザー情報のインジェクション

ダミー情報をインジェクションしているので、ログイン動作をすることなくこのページの動作が確認できます。

# ソース #
```

<?php

/**
 * App
 *
 * @category   BEAR
 * @package    App_Page
 */
require_once 'App.php';

/**
 * トップページ
 *
 * 非ログインならログインフォーム、ログイン済みならエントリー投稿フォームとエントリー一覧がページングされて表示されます。
 * runのオプションでtestのインジェクションをするとダミーユーザーでログイン済みとして実行されます。
 *
 * @package    App
 * @subpackage Page
 * @author     $Author: bear $ <username@example.com>
 * @version    SVN: Release: $Id: index.php 74 2009-07-07 07:00:00Z koriyama $
 */
class Page_Index extends App_Page
{
    /**
     * インジェクション
     */
    public function onInject()
    {
        parent::onInject();
        // ヘッダーサービスを注入
        $this->_header = BEAR::dependency('BEAR_Page_Header');
        //認証プロパティを注入
        $this->_auth = BEAR::dependency('App_Auth')->get();
        // tabプロパティを注入
        $this->injectArg('tab', (isset($_GET['tab']) ? $_GET['tab'] : 'home'));
    }

    /**
     * Testインジェクション
     */
    public function onTestInject()
    {
       parent::onInject();
        $this->_auth = array(
            'user' => array('id' => '6',
                'user_name' => 'dummy user id=6',
                'email' => 'bear@example.com',
                'password' => '4a7d1ed414474e4033ac29ccb8653d9b',
                'image' => '8f14e45fceea167a5a36dedd4bea2543',
                'is_private' => '0',
                'created_at' => '2009-05-27 21:06:07',
                'updated_at' => '2009-06-17 16:20:19',
                'deleted_at' => '0000-00-00 00:00:00'
            'profile' => NULL);
       $this->injectArg('tab', 'entry');
    }

    /**
     * Click - ログアウト
     */
    public function onClickLogout($values)
    {
        BEAR::dependency('App_Auth')->logout();
        $this->_header->redirect('.');
    }

    /**
     * Click - エントリー消去
     *
     * @param BEAR_Click $click 消去エントリーID
     */
    public function onClickDelete($values)
    {
        $values = array('id' => $values, 'user_id' => $this->_auth['user']['id']);
        $params = array('uri' => 'entry', 'values' => $values);
        $this->_resource->delete($params);
    }

    /**
     * Init
     *
     * @param array $args ページ引数
     *
     * @requied tab $argsに必要なキー
     */
    public function onInit(array $args)
    {
        if (!$this->_auth) {
            BEAR::dependency('App_Form_Login')->build(); //ログインフォームのビルト
        } else {
            BEAR::dependency('App_Form_Entry')->build(); //エントリーフォームのビルト
            // 記事リソース取得
            $options['cache']['life'] = 60; //リソースキャッシュ
            $options['pager'] = self::PAGER_NUM; //ページグネーション
            if ($args['tab'] === 'home') {
                $values['user_id'] = $this->_auth['user']['id'];
            } else {
                $values = array();
            }
            $params = array('uri' => 'entry',
                'values' => $values,
                'options' => $options);
            $this->_resource->read($params)->set();
        }
         $this->set('auth', $this->_auth);
    }

    /**
     * Output
     *
     * initでset()した値を出力します。
     * init()では出力ができません。echoや不用意なincludeでの出力はキャンセルされます。
     */
    public function onOutput()
    {
        $this->display(); //画面出力
       //$this->output('json');
       //$this->output('excel');
    }

    /**
     * ログインフォームアクション
     *
     *  id/passでログインリソース->ユーザーリソース(user.id)->プロフィールリソース(profile.user_id)のリンクを辿り情報を一度に取得し、セットします。
     * set()はonOutput()の動作により挙動がきまります。
     * display()ならsmartyテンプレートにアサインした事になり、output('json')ならJSON用データを用意したことになります。
     *
     * @param array $submit
     */
    public function onActionLoginForm(array $submit)
    {
        $params = array('uri' => 'user/login', 'values' => $submit);
        $auth = $this->_resource->read($params)->link('user')->link('profile')->getBody(true);
        //セッションにはユーザー、プロフィールを認証情報としてセット
        $this->_session->set('auth', $auth);
        $this->_header->redirect('.');
    }

    /**
     * OpenIDログインフォームアクション
     *
     * @param array $submit
     */
    public function onActionOpenIdForm(array $submit)
    {
     ...
    }

    /**
     * エントリー投稿
     *
     * @param array $submit
     */
    public function onActionEntryForm(array $submit)
    {
        $submit['user_id'] = $this->_auth['user']['id'];
        $params = array('uri' => 'entry', 'values' => $submit);
        $this->_resource->create($params);
        $this->_header->redirect('.');
    }
}

// initキャッシュとテストインジェクションを指定してページ実行
$config = array('injector'=>'onTestInject', 'cache'=>array('type'=>'init', 'life'=>3));
BEAR_Main::run('Page_Index', $config);
```