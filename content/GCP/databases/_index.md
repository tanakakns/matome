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