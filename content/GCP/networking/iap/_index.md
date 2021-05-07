---
title: "Identity-Aware Proxy"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 3
---

<!--more-->


## 4. Identity-Aware Proxy

[Identity-Aware Proxy](https://cloud.google.com/iap/docs/concepts-overview?hl=ja) （ Cloud IAP ）は、ID とコンテキストを使用して、アプリケーションや VM へのアクセスを保護するサービス。  
実際に利用してみる。

```bash
# App Engine のサンプルアプリケーションを入手して起動する
$ git clone https://github.com/googlecodelabs/user-authentication-with-iap.git
$ cd user-authentication-with-iap/1-HelloWorld # Python/Flask の Web App
$ gcloud app deploy

# 動作確認
$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-01-837dcf92a29f.uc.r.appspot.com
# 上記にアクセスして Hello World が出れば OK
```

以降の設定やアプリケーションの動確は `https://qwiklabs-gcp-01-837dcf92a29f.uc.r.appspot.com` を前提に記載する。  
IAP を Cloud Console を利用して設定する。

1. ナビゲーション メニュー > [セキュリティ] > [Identity-Aware Proxy] をクリック
2. [API を有効にする] をクリック
3. [IDENTITY-AWARE PROXY に移動] をクリック
4. [同意画面を構成] をクリック： Google OAuth 同意画面（Web画面）を作成する
5. [ユーザーの種類] の下の [内部] を選択し、[作成] をクリック
    - 内部： Google Workspace で作成した組織のユーザがスコープ
    - 外部：テストユーザとして追加した Google アカウントがスコープ
6. 下記のように設定
    - アプリケーション名：IAP Example
    - サポートメール：メールアドレス（自動的に入力されている場合も、選択する場合も）
    - アプリケーション ホームページ リンク：アプリの表示に使用した URL： `https://qwiklabs-gcp-01-837dcf92a29f.uc.r.appspot.com`
    - 承認済みドメイン：アプリケーションの URL のホスト名部分： `qwiklabs-gcp-01-837dcf92a29f.uc.r.appspot.com`
    - アプリケーション プライバシー ポリシー リンク：アプリ内のプライバシー ページへのリンク、ホームページへのリンクの末尾に `/privacy` を追加したもの
    - デベロッパーの連絡先情報：適当なメアド
7. [保存] をクリック（認証情報の作成を求めるメッセージが表示されるが、ここでは認証情報を作成する必要はないのでブラウザタブを閉じる）
8. Flex API を無効にするため、Cloud Shell で `gcloud services disable appengineflex.googleapis.com` を実行
9. [Identity-Aware Proxy] に戻ってページを更新すると、保護できるリソースの一覧が表示される
10. 該当レコードの IAP を有効にする
11. サンプルアプリケーションの Web 画面にブラウザでアクセスすると Google AOuth 同意画面が表示されることを確認する
    - ここでは IAP にユーザを登録していないので、まだ認証をパスすることはできない
12. コンソールの [Identity-Aware Proxy] ページに戻り、[App Engine] アプリの左隣にあるチェックボックスをオンにすると、ページの右側にサイドバーが表示される
13. [メンバーを追加] をクリックし、メールアドレスを入力してから、[Cloud IAP/IAP で保護されたウェブアプリ ユーザー] ロールを選択してそのアドレスに割り当てる

次に作成した IAP をテストする。

1. ホームページ アドレスの URL の末尾に `/_gcp_iap/clear_login_cookie` を追加してウェブブラウザでそのアドレスを開く
    - 単に Cookie をクリアしているだけなので操作事態は気にしない（アプリの仕様）
2. 新たに「Google でログイン」画面が表示されるので、先ほど登録したアカウントでログインする

アプリが IAP で保護されると、通過するリクエストに対して IAP がヘッダーを付与する。  
ヘッダ情報を表示するアプリに更新して内容を確認してみる。

```bash
$ cd ~/user-authentication-with-iap/2-HelloUser
$ gcloud app deploy
```

新バージョンのアプリは以下のようなコードでヘッダ情報を取得している。

```python
user_email = request.headers.get('X-Goog-Authenticated-User-Email')
user_id = request.headers.get('X-Goog-Authenticated-User-ID')
```

アプリケーションにアクセスしてみる。（なお、 URL を再表示しているが、アプリのバージョンが変わっただけなので URL に変更はない）

```bash
$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-01-837dcf92a29f.uc.r.appspot.com
```

アプリケーションにアクセスしているユーザのメールアドレスと ID が表示されているのがわかる。  
IAP を無効（ Cloud Console ウィンドウで、ナビゲーション メニュー > [セキュリティ] > [Identity-Aware Proxy] をクリックし、 App Engine アプリの隣にある IAP 切り替えスイッチをクリックして IAP を無効）にしてからアプリのブラウザをリロードすると、メールアドレスと ID は表示されなくなり、 IAP によってヘッダが付与されなくなることがわかる。

現状、このアプリケーションは以下のようなコマンドでも確認できる通り、 **なりすまし** が可能な状態である。

```bash
$ curl -X GET <your-url-here> -H "X-Goog-Authenticated-User-Email: totally fake email"
```

なりすましを防ぐために **暗号検証** を使用する。  
これは、 IAP によって付与される `X-Goog-IAP-JWT-Assertion` ヘッダで実現する。所謂 JWT Token だ。  
暗号検証に対応したアプリに更新する。

```bash
$ cd ~/user-authentication-with-iap/3-HelloVerifiedUser
$ gcloud app deploy
```

このアプリケーションは以下のようなコードで `X-Goog-IAP-JWT-Assertion` ヘッダ を検証している。

```python
def user():
    assertion = request.headers.get('X-Goog-IAP-JWT-Assertion')
    if assertion is None:
        return None, None

    info = jwt.decode(
        assertion,
        keys(),
        algorithms=['ES256'],
        audience=audience()
    )

    return info['email'], info['sub']
```

前回の手順で IAP が無効になっているので、無効の状態で新バージョンのアプリにアクセスし、その後 IAP を有効にしてからアクセスすることで差分を確認しよう。