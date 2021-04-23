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

### コスト

BigQuery のコストは大きく以下の 2 つある。

- 分析料金
    - クエリの処理にかかる費用
    - SQL クエリ、ユーザー定義関数、スクリプト、テーブルをスキャンする特定のデータ操作言語（DML）とデータ定義言語（DDL）のステートメントなど
- ストレージ料金
    - BigQuery に読み込むデータを保存する費用

分析料金はさらに以下に分かれる。

- オンデマンド料金
    - 各クエリによって処理されたバイト数に基づいて課金
    - 処理されるクエリデータは毎月 1 TB まで無料
    - クエリの見積もりは `bq query --dry_run` で読み取られるバイト数を見積もり、Google Cloud 料金計算ツール（[Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator/)）で料金を見積もる
- 定額料金
    - 仮想 CPU であるスロットを購入（スロットを購入すると、クエリの実行に使用できる専用の処理容量を購入したことになる）
    - スロットは以下のコミットメント プランで使用できる
        - Flex Slots: 最初の 60 秒分のコミットメントを購入
        - 月間: 最初の 30 日分のコミットメントを購入
        - 年間: 365 日分のコミットメントを購入
    - 月間プランと年間プランでは、長期的な容量コミットメントと引き換えに割引が適用される

ストレージ料金はさらに以下に分かれる。

- アクティブ
    - テーブルで、過去 90 日間で変更されたデータに対する月額料金
- 長期
    - テーブルまたはパーティションで、過去 90 日間変更されていないデータに対する月額料金