---
title: "App Engine"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 3
---

**App Engine** は Google のインフラストラクチャ上に仮想マシンを構築するサービス。

1. [コンセプト](#1-コンセプト)
2. [インスタンス作成](#2-インスタンス作成)
3. [ディスクの作成とアタッチ](#3-ディスクの作成とアタッチ)
4. [ファイアウォールルールの作成](#4-ファイアウォールルールの作成)
5. [インスタンスへの接続](#3-インスタンスへの接続)
6. [Cloud Monitoring](#4-cloud-monitoring)
7. コマンド整理

<!--more-->

## 1. コンセプト

- [スタンダード環境](https://cloud.google.com/appengine/docs/standard)
    - サンドボックス内で実行される
- [フレキシブル環境](https://cloud.google.com/appengine/docs/flexible)
    - Compute Engine 仮想マシン（VM）上の Docker コンテナ内で実行される

1 リージョン・マルチゾーンで構成される。  
また、 App Engine を作成すると、デフォルトで 単一リージョンで Cloud Storage パケットが作成される。

## 2. App Engine の作成

- Cloud Build API を有効にする
- App Engine アプリを初期化する
    - `gcloud app create --project=[YOUR_PROJECT_ID]`
- ランタイムに対応する [コンポーネント](https://cloud.google.com/appengine/docs/standard/go/quickstart) をインストール
    - `gcloud components install app-engine-go`
- 以下の設定ファイルで App Engine の設定を行う
    - [`app.yaml`](https://cloud.google.com/appengine/docs/standard/go/config/appref) ：オンラインの場合
        - `service1.yaml` `service2.yaml` といった形でサービス単位で分割管理できる
    - [`cron.yaml`](https://cloud.google.com/appengine/docs/standard/go/scheduling-jobs-with-cron-yaml) ：ジョブの場合
    - [`dispatch.yaml`](https://cloud.google.com/appengine/docs/standard/go/reference/dispatch-yaml) ：ルーティング設定
    - [`index.yaml`](https://cloud.google.com/appengine/docs/standard/go/configuring-datastore-indexes-with-index-yaml) ： Datastore の設定
- 各設定ファイルは `gcloud app deploy xxxx.yaml` で適用する
- `gcloud app deploy --no-promote` で新しいバージョンをデプロイしますが、トラフィックが新しいバージョンに自動的にルーティングされない（テストできる）
- `gcloud app logs tail` でログ確認
- 新しいバージョンへのトラフィックの移行する場合、以下の方法がある
    - `gcloud app services set-traffic [MY_SERVICE] --splits [MY_VERSION]=1` ：トラフィックの即時移行
    - `gcloud app services set-traffic [MY_SERVICE] --splits [MY_VERSION]=1 --migrate` ：トラフィックの段階的移行
- また、移行だけでなく、A/B Testing などトラフィック分割も可能
    - `gcloud app services set-traffic [MY_SERVICE] --splits [MY_VERSION1]=[VERSION1_WEIGHT],[MY_VERSION2]=[VERSION2_WEIGHT] --split-by [IP_OR_COOKIE]`