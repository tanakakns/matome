---
title: "Homebrew"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 2
---

Mac のパッケージマネージャである [Homebrew](http://brew.sh/index_ja.html) について。

1. 環境設定  
2. brew tap  
3. brew cask

<!--more-->

## 1. 環境設定

### 1.1. インストール

```zsh
% /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”
==> This script will install:
/usr/local/bin/brew
/usr/local/share/doc/homebrew
/usr/local/share/man/man1/brew.1
/usr/local/share/zsh/site-functions/_brew
/usr/local/etc/bash_completion.d/brew
/usr/local/Homebrew
==> The following new directories will be created:
/usr/local/Cellar
/usr/local/Homebrew
/usr/local/Frameworks
/usr/local/bin
/usr/local/include
/usr/local/lib
/usr/local/opt
/usr/local/sbin
/usr/local/share/zsh
/usr/local/share/zsh/site-functions
```

上記を実行後、 `brew doctor` で問題がないことを確認。  
もしWarningが出たらメッセージに従って対処する。  
筆者の場合はXcodeをインストールしてCommand Line Toolsをアップデートしたら解決した。  
変更があるかもしれないので[公式](http://brew.sh/index_ja.html)を確認のこと。

- `/usr/local/Cellar/`
    - Homebrewでインストールしたソフトウェアはここに配備される
- `/usr/local/bin/`
    - インストールしたソフトウェアのコマンドのシンボリックリンクはここに配備される

### 1.2. Homebrew のアップデート

```zsh
% brew update
```

### 1.3. ソフトウェアのインストール

```zsh
% brew install <software>
```

なお、かつて Homebrew の拡張であった **Cask** は本体に統合された。  
明示的に Cask 管理のソフトウェアをインストールする場合は以下。

```zsh
% brew install <software> --cask
```

### 1.4.ソフトウェアの更新

```zsh
% brew upgrade <software>
```

ソフトウェアを指定しない場合は、インストールしたもの全てが更新される。

### 1.5. キャッシュされている古いソフトウェアの削除

```zsh
% brew cleanup
```

## 2. brew tap

brew tap は、公式以外の formula を追加することのできる Homebrew のサブコマンド。  
Homebrewをインストールした際に標準で組み込まれている。

```zsh
% brew tap <userName>/<repository>
```

上記コマンドを実行すると、GitHub 公開リポジトリ (https://github.com/<userName>/homebrew-<repository>) が参照され、/usr/local/Library/Taps以下にインストールされる。  
`brew tap` コマンドでこれまでにインストールされたソフトウェアの一覧が見れる。  
また、 `brew tap <url>` コマンドでGitHub以外のソフトウェアもインストール可能。

## 3. brew cask

brewやbrew tapでソフトウェアをインストールする際、目的のソフトウェアを動かすには依存するその他のソフトウェアをインストールする必要がある場合がある。  
そんなときbrew caskを使用すると目的のソフトウェアをインストールするだけで依存するソフトウェアもインストールしてくれる。  
さらにGoogle ChromeやAtomといったソフトのインストールも可能となる。

- `/usr/local/Caskroom/`
    - Cask でインストールしたソフトウェアはここに配備される
- `/usr/local/bin/`
    - インストールしたソフトウェアのコマンドのシンボリックリンクはここに配備される

### 3.1. インストール

Homebrew 本体に統合され、インストール不要になった。  
`--cask` オプションで `brew cask` コマンドは代替可能。