---
title: "命名規則"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 3
---

Go の命名規則をまとめる。

1. パッケージ名、ディレクトリ名
2. ファイル名
3. 関数、type、構造体
4. レシーバ名
5. 変数・引数名
6. error変数名
7. インターフェース名

<!--more-->

## 1. パッケージ名、ディレクトリ名

- 完結に、全て小文字かつ 1 単語で
  - 例外としては `_test` サフィックス
- utility パッケージにしない
  - base、util、common、lib、misc など
- パッケージ名を繰り返さない
  - `repository.UserRepository` とか NG
  - `list.NewList` でなく `list.New`
- どうしても難しい場合、ディレクトリ名はケバブケースは可
  - `xxx-xxx`

## 2. ファイル名

- スネークケースで
  - `xxx_xxx`

## 3. 関数、type、構造体

- キャメルケースで
  - 公開する場合はアッパーキャメルケース `XxxYyy`
  - 公開しない場合はローワーキャメルケース `xxxYyy`
- Getter/Setter
  - Go では所謂 Getter には `Get` プレフィックスを付けない
    - 例えば `User` を返したいなら、メソッド名は `GetUser` ではなく `User`
  - Setter の場合は SetUser のように `Set` を付ける
    -  (Go で Setter を作るケースはあまり無いけど)

## 4. レシーバ名

Go には class は無いが、 **型に対してメソッドを定義** できる。  
構造体だけにメソッド定義できるのではなく、型に対して定義できることに注意。  
Go ではメソッドを定義する型を **レシーバ** と呼ぶ。

- 英語 1 文字か 2 文字でなるべく短く命名
  - 型が `Client` であれば `c` 、 `cl` 等
- レシーバ名は必ず統一
  - 場所によって `c` が使われていたり `cl` が使われていたりは NG
- 修飾語を利用しない
  - 例えば `httpClient` ならレシーバ名は `c` 、 `DBCreator` なら `c`
- map 等の存在チェックを行う場合は `ok`
  - `id, ok := users[userID]`
- 命名は基本的にキャメルケースだが、元々略語として浸透している単語は一貫した大文字と小文字を使用
  - `url` ではなく `URL` とか、`http` ではなく `HTTP` を使うとか
  - `ACL`、`API`、`ASCII`、`CPU`、`CSS`、`DNS`、`EOF`、`GUID`、`HTML`、`HTTP`、`HTTPS`、`ID`、`IP`、`JSON`、`LHS`、`QPS`、`RAM`、`RHS`、`RPC`、`SLA`、`SMTP`、`SQL`、`SSH`、`TCP`、`TLS`、`TTL`、`UDP`、`UI`、`UID`、`UUID`、`URI`、`URL`、`UTF8`、`VM`、`XML`、`XMPP`、`XSRF`、`XSS`

## 5. 変数・引数名

- 基本的にレシーバ名と同じ
- ただし、スコープに応じて使い分ける
  - `Config` の変数なら、`c` 、 `conf` 、 `cfg` などで使い分ける

## 6. error変数名

- 基本的に `err`
- 公開する際など複数定義が必要な場合は `Err` プレフィックスを付けて宣言

## 7. インターフェース名

- メソッドが 1 つの場合
  - 原則として「メソッド名 + `er/or`」
    - 例えば `Write` メソッドを持つインターフェースであれば `Writer`
  - 目的語がある場合は「目的語 + 動詞 + `er/or`」
    - 例えば `PrintObject` メソッドを持つインターフェースであれば `ObjectPrinter`
- メソッドが 2 つ以上の場合
  - 原則として慣習がある場合それに従い、慣習が無い場合いい感じの名前を考える
    - `io` パッケージの `ReadCloser` のように、メソッドを連ねて前述の `er` を付けるパターン
    - `Codec` のように、特別なメソッドの組み合わせのときに使われるパターン
      - `Codec` は `EncodeDecoder` だったり他の命名にすることも