---
title: "Go Modules"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 2
---

<!--more-->

Go 1.16 以降。

## go.mod の作成

`go mod init` を使用して、作性するプロジェクトのリポジトリを指定する。

```bash
go mod init github.com/pepese/golang-gin-sample
```

## go.mod/go.sum の更新

Go 1.15 までは `go build` や `go test` などのコマンドで go.mod/go.sum の内容が勝手に更新されていたが， 1.16 からは自動では更新されなくなった。  
なので，コード上の import で新しい外部パッケージを追加しても go.mod/go.sum に追加されない。  
さらに利用のないパッケージなどが残ったりした場合なども含めて、go.mod/go.sum を最適化するのは以下のコマンド。

```bash
$ go mod tidy
```

なお、上記ではパッケージバージョンの最新化などが実施されないため、 `go get` を使って go.mod/go.sum の記述を変更する。

```bash
$ go install golang.org/x/tools/gopls@v0.6.5
```

バージョン番号サフィックスが有効になっており、 `@latest` なども利用できる。

## go install/get

go install/get の違いは以下の通り。

- バイナリのビルドとインストールのための go install
- go.mod 編集のための go get

ツールの導入は `go install` 、go.mod の編集は `go get` を使おう。

## go.mod

go.mod では以下のディレクティブが使用される。

|ディレクティブ|例|説明|
|:---|:---|:---|
|module|module github.com/pepese/golang-gin-sample|モジュール名、 `go mod init` で指定したやつ|
|go|go 1.17|有効な Go バージョン|
|require|require other/thing v1.0.2|インポート・パッケージ|
|exclude|exclude old/thing v1.2.3|除外モジュール|
|replace|replace bad/thing v1.4.5 => good/thing v1.4.5|モジュールの置換|
|retract|retract v1.0.5|撤回バージョン|

### go ディレクティブの更新

```bash
$ go mod tidy -go=1.17
```