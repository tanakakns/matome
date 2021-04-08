---
title: "データベース"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 6
---

<!--more-->

- [Google Cloud データベース](https://cloud.google.com/products/databases)
    - それぞれの違いなど
- リレーショナル（ SQL ）
    - [Cloud SQL](https://cloud.google.com/sql/docs) ：小規模、水平スケーラビリティ不要
    - [Cloud Spanner](https://cloud.google.com/spanner/docs)　：大規模、水平スケーラビリティ、グローバル分散
    - [Bare Metal Solution](https://cloud.google.com/bare-metal/docs)
- Key-Value
    - [Cloud Bigtable](https://cloud.google.com/bigtable/docs) （ HBase 互換 ）：大容量、低レイテンシ（レイテンシが許容されるなら BQ ）
- ドキュメント
    - [Firestore](https://cloud.google.com/firestore/docs) （旧 Datastore ）：トランザクション可能、サーバレス、水平スケーラビリティ
        - [Datastore モード](https://cloud.google.com/datastore/docs?hl=ja)
    - [Firebase Realtime Database](https://firebase.google.com/products/realtime-database/)
- インメモリ
    - [Memorystore](https://cloud.google.com/memorystore/docs) ：キャッシュ、超低レイテンシ、非永続化
- その他
    - [BigQuery](https://cloud.google.com/bigquery/docs?hl=ja) ：大規模なデータアナリティクス

## Cloud SQL

[Cloud SQL](https://cloud.google.com/sql/docs) はクラウド上の PostgreSQL と MySQL のリレーショナル データベースを簡単に設定、維持、運用、管理できるようにするフルマネージド データベース サービス。  
Cloud SQL が対応しているデータ形式には、ダンプファイル（.sql）と CSV ファイル（.csv）。

以下のコマンドで Cloud SQL に接続できる。

```bash
$ gcloud sql connect qwiklabs-demo --user=root
```

## BigQuery

[BigQuery](https://cloud.google.com/bigquery/docs) は Google Cloud Platform 上で動作する、ペタバイト規模のフルマネージド データ ウェアハウス。  
データ アナリストやデータ サイエンティストは、サーバーを設定、管理することなく、大規模なデータセットに対するクエリ（ SQL ）やフィルタの実行、結果の集計、複雑な操作の実行が可能。  
コマンドライン ツール（ `bq` ）またはウェブ コンソールを使用して、GCP プロジェクトに格納されているデータを管理、照会できる。