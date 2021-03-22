---
title: "kubectl plugin と Krew"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 4
---

kubectlのプラグイン機構 と **Krew** についてまとめる。

1. kubectl plugin  
2. Krew

<!--more-->

## 1. kubectl plugin

**kubectl plugin** は kubectl のプラグイン機構で、kubectl に任意のサブコマンドを追加する機能である。  
このプラグイン機構は、 `PATH` の通ったディレクトリに配置された実行ファイルの名前が特定のプレフィックスで開始しているものを呼び出すというもので、kubectl plugin の場合はプレフィックスが `kubectl-` 。  
例えば `kubectl-foo` という実行ファイルを作成し、 `PATH` の通ったディレクトリに配置すると、 `kubectl foo` コマンドとして実行できる。

```zsh
% echo '#!/bin/bash\n\necho "foo"' > /usr/local/bin/kubectl-foo
% chmod +x /usr/local/bin/kubectl-foo
% /usr/local/bin/kubectl-foo
foo
% kubectl foo
foo
```

`kubectl plugin list` コマンドで利用可能なプラグインの一覧を取得できる。

公式：[Extend kubectl with plugins](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/)

## 2. Krew

[Krew](https://github.com/kubernetes-sigs/krew) は、 kubectl plugin をパッケージマネージャ Homebrew 風に管理できるツール。

### 2.1. インストール

[公式](https://krew.sigs.k8s.io/docs/user-guide/setup/install/) を参照するのが望ましいが、ここでは実際にやった手順を残しておく。  

`git` がインストールされていることを確認した後、以下を実行する。

```zsh
% (
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"${OS}_${ARCH}" &&
  "$KREW" install krew
)

（中略）

\
 | Use this plugin:
 |      kubectl krew
 | Documentation:
 |      https://krew.sigs.k8s.io/
 | Caveats:
 | \
 |  | krew is now installed! To start using kubectl plugins, you need to add
 |  | krew's installation directory to your PATH:
 |  | 
 |  |   * macOS/Linux:
 |  |     - Add the following to your ~/.bashrc or ~/.zshrc:
 |  |         export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
 |  |     - Restart your shell.
 |  | 
 |  |   * Windows: Add %USERPROFILE%\.krew\bin to your PATH environment variable
 |  | 
 |  | To list krew commands and to get help, run:
 |  |   $ kubectl krew
 |  | For a full list of available plugins, run:
 |  |   $ kubectl krew search
 |  | 
 |  | You can find documentation at
 |  |   https://krew.sigs.k8s.io/docs/user-guide/quickstart/.
 | /
/
```

`$HOME/.krew/bin` ディレクトリを作成し、パスを通す。（もちろん、プロファイルに記載する）

```zsh
% export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

`kubectl krew` コマンドを実行し、 Usage が表示されてば完了。  
また、 Krew 自身も kubectl plugin なので、 `kubectl plugin list` で表示される。

```zsh
% kubectl plugin list
The following compatible plugins are available:

/Users/pepese/.krew/bin/kubectl-krew
```

アンインストールしたい場合は、 `rm -rf -- ~/.krew` 実行と、環境変数を設定したプロファイルを削除すればいい。

### 2.2. クイックスタート

先に同じく [公式](https://krew.sigs.k8s.io/docs/user-guide/quickstart/) を参照するのが、望ましいがここでは実際にやった手順を残しておく。

```zsh
# プラグインリストの更新
% kubectl krew update

# プラグインの検索
% kubectl krew search
（いっぱい出る）

# プラグイン検索（キーワード指定）
% kubectl krew search ctx
NAME           DESCRIPTION                                      INSTALLED
access-matrix  Show an RBAC access matrix for server resources  no
allctx         Run commands on contexts in your kubeconfig      no
ctx            Switch between contexts in your kubeconfig       no

# プラグインのインストール
% kubectl krew install ctx
Updated the local copy of plugin index.
Installing plugin: ctx
Installed plugin: ctx
\
 | Use this plugin:
 |      kubectl ctx
 | Documentation:
 |      https://github.com/ahmetb/kubectx
 | Caveats:
 | \
 |  | If fzf is installed on your machine, you can interactively choose
 |  | between the entries using the arrow keys, or by fuzzy searching
 |  | as you type.
 |  | See https://github.com/ahmetb/kubectx for customization and details.
 | /
/
WARNING: You installed plugin "ctx" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.

# プラグインの実行
% kubectl ctx

# プラグインの更新
% kubectl krew upgrade
Updated the local copy of plugin index.
Upgrading plugin: ctx
Skipping plugin ctx, it is already on the newest version
Upgrading plugin: krew
Skipping plugin krew, it is already on the newest version

# プラグインのアンインストール
% kubectl krew uninstall ctx
```

## 参考

- [公式ドキュメント](https://krew.sigs.k8s.io/docs/)
- [kubectl のプラグイン機能 kubectl plugin を使おう！](https://qiita.com/superbrothers/items/b4a0aab0575ca6d65739)
- [kubectlのプラグイン機構とおすすめプラグインのご紹介](https://techblog.yahoo.co.jp/entry/2020081830014718/)
- [Krew のカスタムプラグインインデックスで kubectl プラグインを配布しよう](https://qiita.com/superbrothers/items/f2fb422b70ab6f61e531)