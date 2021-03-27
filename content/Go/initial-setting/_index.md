---
title: "初期設定"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 1
---

Go言語の初期設定周りについて。

1. 環境変数
2. Go Modules
3. プロジェクトの作成
4. VS Code の設定
5. docker によるマルチステージビルド
6. デバッグ環境作成

<!--more-->

## 1. 環境変数の設定

https://golang.org/doc/install/source#environment

- `GOROOT`
    - go のバイナリのホームまでのパス
	- `go env GOROOT` で値確認
- `GOPATH`
    - `go env GOPATH` で値確認
    - go の各種資材が配置パスであってプロジェクトのパスでないことに注意
    - プロジェクトのパスは `$GOPATH/src/github.com/<Githubアカウント名>/<プロジェクト名>`
	    - ただし、後述する **GOPATH mode** の場合
- `GOOS`
    - コンパイルして作成するバイナリの対象 OS を指定する
- `GOARCH`
    - コンパイルして作成するバイナリの対象 CPU を指定する

`.zshenv` （ bash の場合は `.bash_profile` ）に以下を追記。

```zsh
export GOROOT=`go env GOROOT`
export GOPATH=`go env GOPATH`
export PATH=$GOPATH/bin:$PATH
# export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
export GO111MODULE=on
```

反映。

```zsh
% exec $SHELL -l
```

また、 `$GOPATH` 以下のディレクトリ構造は以下のようになる。

```zsh
$GOPATH
├─bin/ # Goツール類の実行ファイルが格納されるディレクトリ
├─pkg/ # ビルドしたパッケージオブジェクトが格納されるディレクトリ
│  ├─darwin_amd64/
│  │ ├─github.com/
│  │ │  └─GitHubアカウント名
│  │ │    ├─`*.a`ファイル[^1]
│  │ │    └─GitHubレポジトリ名/`*.a`ファイル[^2]
│  │ └─pkg.in/
│  │   ├─パッケージ名/
│  │   └─`*.a`ファイル
│  └─mod/ # Go Modules で取得したモジュール類が格納されるディレクトリ
│    ├─cache/ # download したモジュール類のメタ情報や実態のzipなどのキャッシュ
│    ├─github.com/ # リポジトリから取得したモジュール
│    └─gopkg.in/ # リポジトリから取得したモジュール
└─src/ # パッケージごとのソースコードを配置するディレクトリ
  ├─gopkg.in/
  │  └─パッケージ名/
  │    └─LICENSEとか`*.go`とかREADMEとか
  └─github.com/
    ├─GitHubアカウント名
    │  └─GitHubレポジトリ名/
    │    └─LICENSEとか`*.go`とかREADMEとか
    └─<あなたのGitHubユーザ名> # Go Modules OFF の場合
      └─GitHubレポジトリ名/ # プロジェクトディレクトリ（複数）
        ├─go.mod
        ├─go.sum
        ├─main.go
        └─その他、あなたが開発中のソフトウェアのコード
```

## 2. Go Modules

Go v1.11 から導入 Go v1.12 から正式リリースされている依存関係管理ツール **Go Modules** （旧名 vgo）の概要について記載する。

- Minimal Version Selection
    - Go Modulesは [Semantic Versioning](https://semver.org/) に基づいてモジュールの管理を行なう
- Go Modulesは **GOPATH mode** と **module-aware mode** という２つのモードがあり、環境変数 `GO111MODULE` で切り替える
    - **GOPATH mode**
	    - `$GOPATH/src` 配下で `go get` コマンドを利用した依存性管理（ go v1.10 以前と同じ）
	    - 標準 pkg 以外を全部 `$GOPATH/src` 以下のディレクトリで管理する
		- `$GOPATH/src/github.com` 配下に普通に `git clone` した状態のモジュールを参照する
		    - `go get` で `go1` タグ・ブランチもしくは最新の `master` ブランチを取得したもの
    - **module-aware mode**
	    - `go mod` コマンド・ `go.mod` ファイルを利用した依存性管理（ `go.sum` ：依存モジュールのチェックサムの管理）
	    - 標準 pkg 以外の全てのパッケージをモジュールとして管理する
		- GOPATH mode とは異なり、 `$GOPATH/pkg/mod` 配下に同じモジュールでも Semantic Versioning された単位で管理され `go.mod` に記載されたバージョンを参照する
	- 環境変数 `GO111MODULE`
		- `GO111MODULE=off` ： **GOPATH mode**
		- `GO111MODULE=on` ： **module-aware mode**
		- `GO111MODULE=auto` ： `$GOPATH/src` 配下では GOPATH mode 、それ以外では module-aware mode で動作する

従来通り `$GOPATH` 配下でプロジェクトを作成・開発し且つ module-aware mode を利用したい場合には `GO111MODULE=on` とする必要がある。  
また、 `$GOPATH` 配下以外でプロジェクトを作成する場合であっても module-aware mode を利用すれば `$GOPATH/pkg/mod` 配下で依存モジュールが管理される。

## 3. プロジェクトの作成

`$GOPATH` 配下で github および Go Modules を利用したプロジェクトの初回作成は以下のような感じ。

```zsh
% mkdir -p $GOPATH/src/github.com/<あなたのGithubアカウント名>
% cd $GOPATH/src/github.com/<あなたのGithubアカウント名>
% mkdir <golangプロジェクト> # プロジェクトディレクトリの作成
% cd <golangプロジェクト>
% export GO111MODULE=on
% go mod init              # `GO111MODULE=on go mod init` のように一行でも
go: creating new go.mod: module github.com/<あなたのGithubアカウント名>/<golangプロジェクト>
% ls
go.mod
% touch app.go             # 依存ライブラリ含め好きなコード書く
% go mod tidy              # 依存ライブラリの解決
% go run app.go            # ビルド+実行
% go build app.go          # ビルド
% ./app
```

基本的にはどこのディレクトリで開発しようとも **module-aware mode** で開発することになると思う。

## 4. VS Code の設定

VS Code には Go の各種ツールと連携する拡張機能があり、 VS Code 内ターミナルから以下のようにコマンドラインツールを導入することにより自動で拡張機能インストールの案内をしてくれる。  
Go の拡張機能をインストールしてから、VSCode でコマンドパレット(Cmd+Shift+P)を開いて `GO: Install/Update tools` で検索した後、全チェックしてインストールしてもいいし、以下で 1 つずつ入れてもいい。

- goimports
    - 過不足のimportの自動補完
	- `go get golang.org/x/tools/cmd/goimports`
- gocode
    - ヘルパー機能
	- `go get -u -v github.com/nsf/gocode`
- godef
    - 呼び出し関数へのジャンプなど
	- `go get -u -v github.com/rogpeppe/godef`
- gogetdoc
    - `go get -u -v github.com/zmb3/gogetdoc`
- golint
    - lint
	- `go get -u -v golang.org/x/lint/golint`
- go-outline
    - `go get -u -v github.com/lukehoban/go-outline`
- goreturns
    - `go get -u -v sourcegraph.com/sqs/goreturns`
- gorename
    - `go get -u -v golang.org/x/tools/cmd/gorename`
- gopkgs
    - `go get -u -v github.com/tpng/gopkgs`
- go-symbols
    - `go get -u -v github.com/newhook/go-symbols`
- guru
    - `go get -u -v golang.org/x/tools/cmd/guru`
- gotests
    - `go get -u -v github.com/cweill/gotests/...`

## 4.1. .editorconfig

```
root = true

[*]
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = false
charset = utf-8
indent_style = space
indent_size = 2

[{Makefile,go.mod,go.sum,*.go}]
indent_style = tab
indent_size = 4
trim_trailing_whitespace = true
```

## 5. docker によるマルチステージビルド

```
FROM golang:1.13.3-alpine3.10 as build
WORKDIR /build
COPY go.mod .
COPY go.sum .
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go

FROM alpine:3.10
RUN apk add --no-cache tzdata
ENV TZ Asia/Tokyo
COPY --from=build /build/main /usr/local/bin/main
EXPOSE 8080
ENTRYPOINT ["/usr/local/bin/main"]
```

## 6. デバッグ環境作成

デバッガとして **Delve** を導入する。

- デバッガツール delve のインストール
    - `go get -u github.com/derekparker/delve/cmd/dlv`
- VSCodeにGo言語の拡張機能をインストール
    - `Rich Go language support for Visual Studio Code`

コードにブレークポイントを設定して、 VSCode の `Debug` から `Start Debugging` を実行。（ `F5` でも）  
以下を実行可能。

- 継続実行（Continue）
    - 次のブレークポイントに到達するまで処理を継続させる
- ステップオーバー（Step Over）
    - 見えているソースコードの次の行に移動する。 カーソル位置の関数の中は実行され、終了するところまで処理が進む
- ステップイン（Step Into）
    - 関数呼び出しの中に飛び込む。下のレイヤーに降り ていくときに使う
- ステップアウト（Step Out）
    - いま実行している関数が終了するところまで処理 を進める
- 再スタート（Restart）
    - 一度終了して再度実行を開始する
- 停止（Stop）
    - 一度終了する

例えば `fmt.Println()` をステップインして細かいところを見ていこう。  
`syscall.Write()` などでシステムコールされているのがわかる。（ Win だと別コード）  
今回はデバッガのステップインでコードを掘っていったが、カーソルがあたっている位置の関数で `Go to Definition` （ `F12` ）しても関数の定義に飛ぶ。  
また、カーソルがあたっている位置の関数や変数で `Find All Reference` （ `Shift+F12` ） すれば、使われている位置がリストされる。