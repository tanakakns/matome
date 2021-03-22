---
title: "asdf"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 3
---

[asdf](https://asdf-vm.com/#/)は、プログラミング言語やコマンドラインツールのバージョン管理をツール。（2020/12/09時点ではanyenvより対応が多い）

1. インストール・アップデート・アンインストール  
2. 使い方

<!--more-->

## 1. インストール・アップデート・アンインストール

基本は[公式](https://asdf-vm.com/#/core-manage-asdf)を見て欲しいが、ここでは本ブログ作成日におけるMac環境へのインストールについて記載する。

### 1.1. インストール

```zsh
% brew install coreutils curl git
% brew install asdf
% echo -e "\n. $(brew --prefix asdf)/asdf.sh" >> ~/.zshenv
```

### 1.2. アップデート

```zsh
% brew upgrade asdf
```

### 1.3. アンインストール

```zsh
% rm -rf ${ASDF_DATA_DIR:-$HOME/.asdf} ~/.tool-versions
```

プロファイルに設定した asdf へのパス削除を忘れずに。

## 2. 使い方

### 2.1. プラグイン・ツールのインストール

1. 使いたいプログラミング言語・ツールに対応したプラグインを探す
    - `asdf plugin list all`
2. 使いたいプログラミング言語・ツールに対応したプラグインを追加する
    - `asdf plugin add <name>`
    - `asdf plugin add <name> <git-url>` # asdf で管理されていないプラグインの場合
3. プラグインが追加されたか確認する
    - `asdf plugin list`
4. 使いたいプログラミング言語・ツールのバージョンを探す
    - `asdf list-all <name>`
5. 使いたいプログラミング言語・ツールのバージョンを指定してインストールする
    - `asdf install <name> <version>`
6. global/local/shell のいずれかでプログラミング言語・ツールのバージョンを設定する
    - `asdf global <name> <version>`
7. 現在のバージョンを確認する
    - `asdf current`
    - `asdf current <name>`

helm で一連の流れをやってみる。

```zsh
% asdf plugin list all | grep helm
helm                          https://github.com/Antiarchitect/asdf-helm.git
helm-cr                       https://github.com/Antiarchitect/asdf-helm-cr.git
helm-ct                       https://github.com/tablexi/asdf-helm-ct.git
helm-docs                     https://github.com/sudermanjr/asdf-helm-docs.git
helmfile                      https://github.com/feniix/asdf-helmfile.git

% asdf plugin add helm

% asdf plugin list
helm

% asdf list-all helm
3.4.1

% asdf install helm 3.4.1

% asdf global helm 3.4.1

% asdf current helm
helm            3.4.1           /Users/xxxx/.tool-versions

% helm version
version.BuildInfo{Version:"v3.4.1", GitCommit:"c4e74854886b2efe3321e185578e6db9be0a6e29", GitTreeState:"clean", GoVersion:"go1.14.11"}
```

### 2.2. プラグインのアップデート

```zsh
% asdf plugin update --all
```

### 2.3. ツールのアンインストール

```zsh
% asdf uninstall <name> <version>
```

### 2.4. プラグインのアンインストール

```zsh
% asdf plugin remove <name>
```

### 2.5. 困ったときは

```zsh
% asdf help
```