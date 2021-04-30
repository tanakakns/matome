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

Cloud SQL、Cloud Spanner、Bare Metal Solution、Cloud Bigtable、Firestore、Memorystore、BigQuery

<!--more-->

[Google Cloud データベース](https://cloud.google.com/products/databases)

|サービス|モデル|提供|ユースケース|
|:---|:---|:---|:---|
|[Cloud SQL](./cloud-sql) ※[doc](https://cloud.google.com/sql/docs)|リレーショナル（ SQL ）|インスタンス|小規模（TB級）、水平スケーラビリティ不可|
|[Cloud Spanner](./cloud-spanner) ※[doc](https://cloud.google.com/spanner/docs)|リレーショナル（ SQL ）|インスタンス？|大規模（PB級）、水平スケーラビリティ、グローバル分散|
|[Bare Metal Solution](./bare-metal-solution) ※[doc](https://cloud.google.com/bare-metal/docs)||||
|[Cloud Bigtable](./cloud-bigtable) ※[doc](https://cloud.google.com/bigtable/docs)|Key-Value（ HBase 互換）||大容量（PB級）、超低レイテンシ（レイテンシが許容されるなら BQ ）|
|[Firestore](./firestore) ※旧 [Datastore](https://cloud.google.com/datastore/docs?hl=ja) ※[doc](https://cloud.google.com/firestore/docs)|ドキュメント|サーバレス|TB級、トランザクション可能、サーバレス、水平スケーラビリティ|
|[Firebase Realtime Database](./firebase-realtime-database) ※[doc](https://firebase.google.com/products/realtime-database/)||||
|[Memorystore](./memorystore) ※[doc](https://cloud.google.com/memorystore/docs)|インメモリ||キャッシュ、超低レイテンシ、非永続化|
|[BigQuery](./bigquery) ※[doc](https://cloud.google.com/bigquery/docs?hl=ja)|DWH（ SQL ）||大規模（PB級）なデータアナリティクス|

## データ移行

- gsutil ：オンプレミスデータの転送量が少ない場合
    - Cloud Storage バケットを作成し、データコピー
    - オンプレミスもしくは別の Cloud Storage リージョンからの転送 **1 TB 未満** の場合
- [Storage Transfer Service](https://cloud.google.com/storage-transfer/docs/overview?hl=ja) ：オンプレミスデータの転送量が多い場合
    - 他のクラウドまたはオンプレミスストレージから Cloud Storage バケットにデータを転送またはバックアップするサービス
    - 別の Cloud Storage リージョンからの転送 **1 TB 超** の場合
    - 別のクラウドストレージからの転送の場合
    - オンプレミスから 1 TB 超のデータを転送する場合は [**Transfer service for on-premises data**](https://cloud.google.com/storage-transfer/docs/on-prem-overview?hl=ja)
- Transfer Appliance ：大規模な転送の場合
    - 大量のデータ（数百テラバイトから 1 ペタバイトまで）を Google Cloud Platform に安全に移行できるハードウェア アプライアンス
    - トータルの所用期間は 50 日
- BigQuery Data Transfer Service ：あらかじめ設定されたスケジュールに基づき、BigQuery へのデータの移動を自動化するマネージド サービス

## データ分析パイプライン

「取り込み」 -> 「処理」 -> 「分析」 -> 「可視化」の順で処理をする。

- 取り込み
    - Cloud Pub/Sub
- 処理
    - Cloud Dataflow（推奨）： Apache Beam 準拠の ETL 処理、サーバレス
    - Cloud Dataproc：マネージド Hadoop/Spark、クラスタ管理が必要、既存の Hadoop/Sparkジョブがある場合はこちらでも
- 分析
    - BigQuery
- 可視化
    - DataPortal