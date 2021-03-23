---
title: "初期設定"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 1
---

Gitの認証周りの初期設定について。

1. アカウント
2. 認証
3. Github の認証

<!--more-->

## 1. アカウント

アカウント情報の設定を行う。  
なお、複数アカウント存在する場合は、 `git config --local` での設定を忘れないこと。

### 1.1. グローバル

git コマンドに対するグローバルな設定。  
なお、ローカル設定がある場合はそちらが優先される。

```zsh
% git config --global user.name <githubアカウント>
% git config --global user.email <メアド>
% git config --global http.sslverify false

# 確認
% git config --global --list
```

### 1.2. ローカル

`git clone` した単位で有効な設定。  
設定されていない項目はグローバルもしくはデフォルトが有効になる。

```zsh
% git config --local user.name <githubアカウント>
% git config --local user.email <メアド>
% git config --local http.sslverify false

# 確認
% git config --local --list
```

## 2. 認証

git コマンドを打つ度に認証情報を聞かれるのは煩わしい。  
ここでは認証情報の設定方法について記載する。  
なお、 ssh ではなく https 前提とする。

2.1. osxkeychain（Macの場合）  
2.2. Git-Credential-Manager-for-Windows（Windowsの場合）  
2.3. .netrc（Linuxの場合）

### 2.1. osxkeychain

`credential helper` の `osxkeychain helper` を利用する方法。  
`git credential-osxkeychain` が使えるかを確認する。

```zsh
% git credential-osxkeychain
usage: git credential-osxkeychain <get|store|erase>
```

上記のように表示されれば利用できる。  
以下でヘルパーの設定を行う。

```zsh
% git config --global credential.helper osxkeychain
```

上記は `git clone` するときの `https://<username>@<domain>/~` 単位で認証情報を記憶してくれるため、 `git clone` する際は以下のようにアカウント情報を付与すること。

```zsh
% https://<username>@<domain>/<repository-username>/<repository-name>.git
```

Mac では認証情報は「キーチェーンアクセス」に保存される。

### 2.2. Git-Credential-Manager-for-Windows

Windows の場合は `Git-Credential-Manager-for-Windows` を利用する。

```dos
choco install git-credential-manager-for-windows
```

初回パスワード入力時に Internet Explorer などと同じ場所にアクセストークンが保存され、以後はパスワード入力が不要になる。

### 2.3. .netrc

パスワードを素で記述するためいい方法ではないが、一応書いておく。

```zsh
% cat >> ~/.netrc <<EOF
machine github.com
login <githubアカウント>
password <パスワード>
EOF
% chmod 600 ~/.netrc
```

## 3. Github の認証

Github は 2021 年 8 月 13 日より Basic 認証による Git アクセスが廃止となる。（ [参考](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/) ）  
そのため、個人アクセストークンの設定が必要となる。（ [参考](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/) ）

### 3.1. 個人アクセストークン

1. Web で [Github](https://github.com) にログインする
2. [設定画面](https://github.com/settings/tokens)を開く
3. `Generate new token` をクリック
4. noteにトークンのわかりやすい名前を記入
5. 権限を設定 （ `repo` だけ選択しとけばよい）
6. [Generate token] をクリック
7. トークンをコピー （ページを離れると参照不可になるので注意）
8. パスワードの代わりに使用する