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
|[Cloud SQL](./cloud-sql) ※[doc](https://cloud.google.com/sql/docs)|リレーショナル（ SQL ）|インスタンス|小規模、水平スケーラビリティ不可|
|[Cloud Spanner](./cloud-spanner) ※[doc](https://cloud.google.com/spanner/docs)|リレーショナル（ SQL ）|インスタンス？|大規模、水平スケーラビリティ、グローバル分散|
|[Bare Metal Solution](./bare-metal-solution) ※[doc](https://cloud.google.com/bare-metal/docs)||||
|[Cloud Bigtable](./cloud-bigtable) ※[doc](https://cloud.google.com/bigtable/docs)|Key-Value（ HBase 互換）||大容量、低レイテンシ（レイテンシが許容されるなら BQ ）|
|[Firestore](./firestore) ※旧 [Datastore](https://cloud.google.com/datastore/docs?hl=ja) ※[doc](https://cloud.google.com/firestore/docs)|ドキュメント||トランザクション可能、サーバレス、水平スケーラビリティ|
|[Firebase Realtime Database](./firebase-realtime-database) ※[doc](https://firebase.google.com/products/realtime-database/)||||
|[Memorystore](./memorystore) ※[doc](https://cloud.google.com/memorystore/docs)|インメモリ||キャッシュ、超低レイテンシ、非永続化|
|[BigQuery](./bigquery) ※[doc](https://cloud.google.com/bigquery/docs?hl=ja)|DWH（ SQL ）||大規模なデータアナリティクス|
