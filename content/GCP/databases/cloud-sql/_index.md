---
title: "Cloud SQL"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

<!--more-->

[Cloud SQL](https://cloud.google.com/sql/docs) はクラウド上の PostgreSQL と MySQL のリレーショナル データベースを簡単に設定、維持、運用、管理できるようにするフルマネージド データベース サービス。  
**GCP 外からもアクセス可能** なのが売りの 1 つらしい。  
Cloud SQL が対応しているデータ形式には、ダンプファイル（.sql）と CSV ファイル（.csv）。

- メモ
    - Cloud SQL の構築は ゾーン を指定するものの、VPC やサブネットは指定できないみたい -> ゾーンリソース？
    - Cloud SQL を構築した後、 AUTHORIZATION の項目で、アクセスする VM インスタンスの **グローバル IP** を許可するリストに加える
    - VM インスタンスから Cloud SQL にアクセスするには Cloud SQL の IP アドレス、ユーザ、パスワードを指定する
    - グローバル IP だけでなく、 プライベート IP もサポートされている


## インスタンスの作成

[インスタンスの作成](https://cloud.google.com/sql/docs/mysql/create-instance#gcloud)

```bash
$ gcloud sql instances create INSTANCE_NAME --cpu=NUMBER_CPUS --memory=MEMORY_SIZE --region=REGION
```

## Cloud Shell から `gcloud` でインスタンスに接続

```bash
$ gcloud sql connect qwiklabs-demo --user=root
```

## Cloud SQL Auth Proxy

- Cloud SQL Auth Proxy はローカル環境から Cloud SQL へ接続するための仕組み
- テスト用
- ローカル PC に 「Cloud SQL Auth Proxy クライアント」を導入して、 `gcloud` で認証することにより接続できる
- [Cloud SQL Auth Proxy を使用するためのクイックスタート](https://cloud.google.com/sql/docs/mysql/quickstart-proxy-test)
