---
title: "BigQuery"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

<!--more-->

[BigQuery](https://cloud.google.com/bigquery/docs) は Google Cloud Platform 上で動作する、ペタバイト規模のフルマネージド データ ウェアハウス。  
データ アナリストやデータ サイエンティストは、サーバーを設定、管理することなく、大規模なデータセットに対するクエリ（ SQL ）やフィルタの実行、結果の集計、複雑な操作の実行が可能。  
コマンドライン ツール（ `bq` ）またはウェブ コンソールを使用して、GCP プロジェクトに格納されているデータを管理、照会できる。

1. アクセスコントロール
1. パーティション分割と有効期限
2. 外部データソースに対するクエリ
3. コスト

## 1. アクセスコントロール

[BigQuery の IAM 事前定義ロール](https://cloud.google.com/bigquery/docs/access-control#bigquery)

- BigQuery 管理者 `roles/bigquery.admin`
    - 全権限
- BigQuery データ編集者 `roles/bigquery.dataEditor`
    - テーブル・ビュー・データ・データセットの CRUD
    - ジョブに関する権限は無い
- BigQuery データオーナー `roles/bigquery.dataOwner`
    - BigQuery データ編集者 `roles/bigquery.dataEditor` の上位互換？
    - ジョブに関する権限は無い
- BigQuery データ閲覧者 `roles/bigquery.dataViewer`
    - テーブル・ビュー・データ・データセットの Read のみ
    - ジョブに関する権限は無い
- BigQuery ジョブユーザー `roles/bigquery.jobUser`
    - ジョブの作成・実行権限
    - ジョブを通じてのみデータへアクセスできる（直接データを閲覧sうる権限は無い）
- BigQuery メタデータ閲覧者 `roles/bigquery.metadataViewer`
    - メタデータ（テーブル・ビュー・データセット） の Read のみ

## 2. 有効期限

BigQuery では以下のようにデータのパーティション分割と有効期限を設定できる。

- [パーティション分割](https://cloud.google.com/bigquery/docs/partitioned-tables#ingestion_time)
    - 取り込み時間パーティション分割テーブル
    - 日付、タイムスタンプ、日時パーティション分割テーブル
- [有効期限](http://cloud.google.com/bigquery/docs/best-practices-storage#use_the_expiration_settings_to_remove_unneeded_tables_and_partitions)
    - パーティションやテーブルデータの有効期限を設定できる

## 3. 外部データソースに対するクエリ

BigQuery は [外部データソース](https://cloud.google.com/bigquery/external-data-sources) に対してクエリを発行できる。

- Cloud SQL
- Bigtable
- Cloud Storage
- Google ドライブ

ただし、データの生合成が取れないなどの制約はある。

## 4. コスト

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