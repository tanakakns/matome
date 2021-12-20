---
title: "クリーンアーキテクチャ"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

<!--more-->

## 概念図

![CleanArchitecture1](https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)

![CleanArchitecture2](https://camo.qiitausercontent.com/90a79f885ff442a5d076cbf8c9dac47b6be29ff0/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3239333336382f33656631653930302d386537342d633935342d326133332d3466343663373536393234342e6a706567)

## 各階層

各層の日本語での呼び方は独自。

### 黄： **ドメイン層** （ Enterprise Business Rules ）

- ドメインのビジネスルールをカプセル化する層
- ユースケース層によって扱われるドメインモデルとドメインロジックを実装する
  - ドメインエンティティは、メソッドを持ったオブジェクト、あるいは、データ構造と関数の集合など
- ユースケースを実行するアプリケーションレベルのビジネスロジックはユースケース層に実装するが、ドメイン固有のビジネスルール・ロジックはここに実装する

### 赤： **ユースケース層** （ Applications Business Rules ）

- アプリケーション固有のビジネスルールを含む
- システムのユースケースすべてをカプセル化および実装する <-> ドメイン層を除く他の層への参照が無く知らない
- ドメイン・アプリケーションレベルのビジネスルールを使って、ユースケースの目的を達成する処理を記述する
- このレイヤーの変更は、ドメイン層には影響を与えない
- このレイヤーは、アダプタ層を介することにより、データベース、UIあるいは、共通のフレームワークの変更から影響を受けない

### 緑： **アダプタ層** （ Interface Adapters ）

- インフラストラクチャ層 と ユースケース層 を相互に繋ぐ層
- プレゼンター、ビュー、そしてコントローラーはなどがここに属す
- 入ってくる方向（ IN ）と出ていく方向（ OUT 、 RETURN ）があり、各必要に応じてデータ変換する
  - Controllers ： [ IN ] コントローラからユースケースにモデルが渡される
  - Gateways ： [ OUT ] ユースケース・ドメイン層が扱うデータ形式から、インフラストラクチャ層が扱うデータ形式に、データ変換する
  - Presenters ： [ RETURN ] ユースケースからプレゼンターやビューにモデルが戻される
- この円よりも内側のコードは、インフラストラクチャ層についてなにも知らない
  - ある Gateways の先のインフラストラクチャ層が RDB であれ Redis であれ関係なく、永続化される程度のことしか知らない

### 青： **インフラストラクチャ層** （ Frameworks & Drivers ）

- データベースやウェブのような外部の機能を実装し、外の世界とシステムを相互に繋ぐ層
- ひとつ内側の円と通信するつなぎのコードが含まれる
- 最も具体的で詳細な実装・変更が多いが多くのコードは書かない
- 一般にフレームワークやツールから構成される
  - データベースやウェブフレームワークなど

## ディレクトリ階層・ファイル例

Golang の場合のディレクトリ構成例。

- **domain** / 黄：ドメイン層（ Enterprise Business Rules ） ： ドメインロジックを入れる
  - **model** ： ドメインモデルを入れる（ドメインロジックと分けなくてもいい）
- **usecase** / 赤：ユースケース層（ Applications Business Rules ）
- **adapter** / 緑：アダプター層（ Interface Adapters ）
  - **controller** ： [ IN ] infrastructure （ Web や API ）からの呼び出しを usecase にマッピングする **handler** ・ **router** を実装する
  - **gateway** ： [ OUT ] infrastructure の datastore や httpclient のインターフェースとなる **repository** をインターフェースとして定義する、ロジックの実装はない
  - **presenter** ： [ RETURN ] ビュー（ HTML や JSON ）へデータを返却する際のインターフェースとして定義する
- **infrastructure** / 青：インフラストラクチャ層（ Frameworks & Drivers ）
  - **server** ： [ IN / RETURN ] FW に依存したリクエスト受信・返却の実装、 view（HTMLとか）、api(JSONとかXML)
  - **batch** ： [ IN / RETURN ] FW に依存したバッチ・ジョブの実装
  - **datastore** ： [ OUT ] データベース製品に依存したアクセス、 repository の実装
  - **httpclient** ： [ OUT ] 他のマイクロサービスが外部サービスの API 呼び出し、 repository の実装

その他のディレクトリとして以下のようなものがあり得る。

- **registry** ： DI とか
- **cmd** ： cobra とかで作ったコマンド
- **view** ： html template とか
- **public** ： css とか js とか

例えば Hello Go は以下のようになる。

```bash
hello-project
│
├── domain
│   ├── hello_logic.go # hello ドメインロジック
│   └── model
│       └── hello.go   # hello ドメインモデル
│
├── usecase
│   └── hello_usecase.go # hello ユースケース
│
├── adapter
│   ├── controller
│   │   └── router.go           # ユースケースへの処理の振り分け、データ変換
│   ├── gateway
│   │   └── hello_repository.go # 永続化層処理の呼び出し、データ変換
│   └── presenter
│       └── hello_view.go       # ユースケース処理後データのビュー変換
│
├── infrastructure
│   ├── server
│   │   └── httpserver.go      # router.go 、 hello_view.go の実装
│   └── datastore
│       └── hello_datastore.go # hello_repository.go の実装
│
└── main.go
```