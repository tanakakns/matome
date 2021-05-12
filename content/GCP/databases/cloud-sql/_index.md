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

[Cloud SQL](https://cloud.google.com/sql/docs) はクラウド上の PostgreSQL 、 MySQL 、 SQL Server のリレーショナル データベースを簡単に設定、維持、運用、管理できるようにするフルマネージド データベース サービス。  
**GCP 外からもアクセス可能** なのが売りの 1 つらしい。  
Cloud SQL が対応しているデータ形式には、ダンプファイル（.sql）と CSV ファイル（.csv）。

- 特徴
    - Cloud SQL の構築は ゾーン を指定するものの、VPC やサブネットは指定できない -> **ゾーンリソース** -> ゾーン障害に対応できない
        - 同一リージョン・別ゾーンに対して **フェールオーバーレプリカ** を作成できる（別リージョンは無理）
    - Cloud SQL を構築した後、 AUTHORIZATION の項目で、アクセスする VM インスタンスの **グローバル IP** を許可するリストに加える
    - VM インスタンスから Cloud SQL にアクセスするには Cloud SQL の IP アドレス、ユーザ、パスワードを指定する
    - グローバル IP だけでなく、 プライベート IP もサポートされている


## インスタンスの作成

- [インスタンスの作成](https://cloud.google.com/sql/docs/mysql/create-instance#gcloud)
- [`gcloud sql instances create`](https://cloud.google.com/sdk/gcloud/reference/sql/instances/create)
- [`gcloud beta sql instances create`](https://cloud.google.com/sdk/gcloud/reference/beta/sql/instances/create)

```bash
$ gcloud sql instances create INSTANCE_NAME --cpu=NUMBER_CPUS --memory=MEMORY_SIZE --region=REGION
```

- **高可用性** インスタンスを作成する場合は、`--zone`と `--secondary-zone` オプションを使用して、プライマリゾーンとセカンダリゾーンの両方を指定できる
- パブリックアクセスする場合は `--authorized-networks=192.168.100.0/24` のように許可する IP アドレスを指定する必要がある
- プライベートアクセスする場合は `--network=projects/testproject/global/networks/testsharednetwork` のように VPC を指定する必要がある
    - ただし、ベータ版コマンド（ `gcloud beta sql instances create`）でのみ有効

## Cloud Shell から `gcloud` でインスタンスに接続

```bash
$ gcloud sql connect qwiklabs-demo --user=root
```

## Cloud SQL Auth Proxy

- Cloud SQL Auth Proxy はローカル環境から Cloud SQL へ接続するための仕組み
- テスト用
- ローカル PC に 「Cloud SQL Auth Proxy クライアント」を導入して、 `gcloud` で認証することにより接続できる
- [Cloud SQL Auth Proxy を使用するためのクイックスタート](https://cloud.google.com/sql/docs/mysql/quickstart-proxy-test)

## ユースケース

### VM インスタンスから Cloud SQL に接続する

- `default` ネットワークの `us-central1-a` ゾーンに `blog` VM インスタンスがある
    - 内部 IP `10.128.0.3` 、 外部 IP `35.194.61.180`
- `blog` VM インスタンスは 1 台で LAMP 構成になっている
- MySQL を `blog-db` という名前の Cloud SQL に移行して接続する
    - `blog` の `/var/www/html/wordpress/wp-config.php` を書き換えて DB の接続先を変更する

```bash
# Cloud Shell ：リージョン・ゾーンの設定
$ gcloud config set compute/region us-central1
$ gcloud config set compute/zone us-central1-a

# Cloud Shell ： Cloud SQL の作成
$ gcloud sql instances create blog-db --region=us-central1 --authorized-networks=35.194.61.180/32


# blod VM インスタンス： blod VM インスタンスへ SSH して mysql ダンプを取得
$ mysqldump --databases wordpress -h localhost -u blogadmin -pPassword1* \
    --hex-blob --single-transaction \
    --default-character-set=utf8mb4 > dump.sql

# blod VM インスタンス： Cloud Storage バケット作成
$ gsutil mb gs://qwiklabs-gcp-01-9aec32589768

# blod VM インスタンス： ダンプをアップロード
$ gsutil cp dump.sql gs://qwiklabs-gcp-01-9aec32589768/

# blod VM インスタンス： アップロードを確認
$ gsutil ls gs://qwiklabs-gcp-01-9aec32589768/
gs://qwiklabs-gcp-01-9aec32589768/dump.sql

# Cloud Console にて blog-db から gs://qwiklabs-gcp-01-9aec32589768/dump.sql をインポート

# blod VM インスタンス： 試しに VM インスタンスから接続
$ mysql -h 35.239.119.80 -u root

# blod VM インスタンス： 設定ファイルを編集した後、 Apache リスタート
$ sudo service apache2 restart

# Cloud Shell ：接続テスト
$ curl http://35.194.61.180
```