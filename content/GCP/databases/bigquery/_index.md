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
2. パーティション分割と有効期限
3. 外部データソースに対するクエリ
4. BigQuery へのデータの読み込み
5. BiGQuery へのクエリ
6. テーブルスキーマの変更
7. コスト

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

|Capability| **dataViewer** | **dataEditor** | **dataOwner** | **metadataViewer** | **user** | **jobUser** | **admin** |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|List/Get projects|o|o|o|o|o|o|o|
|List tables|o|o|o|o|o||o|
|Get table metadata|o|o|o|o|||o|
|Get table data|o|o|o||||o|
|Create tables||o|o||||o|
|Modify/Delete tables||o|o||||o|
|Get dataset metadata|o|o|o|o|o||o|
|Create new datasets||o|o||o||o|
|Modify/Delete datasets|||o||Self||o|
|Create jobs/queries|||||o|o|o|
|Get jobs||||||Self|Any|
|List jobs|||||Any||Any|

なお、リソースに対する権限コントロールの最小は **データセット** となる。テーブル・ビュー・行・列単位の権限コントロールはできない。

## 2. パーティション分割と有効期限

BigQuery では以下のようにデータのパーティション分割と有効期限を設定できる。

- [パーティション分割](https://cloud.google.com/bigquery/docs/partitioned-tables#ingestion_time)
    - 取り込み時間パーティション分割テーブル
    - 日付、タイムスタンプ、日時パーティション分割テーブル
- [有効期限](http://cloud.google.com/bigquery/docs/best-practices-storage#use_the_expiration_settings_to_remove_unneeded_tables_and_partitions)
    - パーティションやテーブルデータの有効期限を設定できる

[パーティション分割テーブルのクエリ](https://cloud.google.com/bigquery/docs/querying-partitioned-tables?hl=ja) を実行したい場合、 疑似列 `_PARTITIONTIME` / `_PARTITIONDATE` が使用でき、適切にパーティションに対してクエリできる。  
さらに、 「From `bigquery-public-data.noaa_gsod.gsod*`」 のようなワイルドカードを利用したテーブルに対しては、 疑似列 `_TABLE_SUFFIX` を利用することでその範囲を指定できる。

## 3. 外部データソースに対するクエリ

BigQuery は [外部データソース](https://cloud.google.com/bigquery/external-data-sources) に対してクエリを発行できる。

- Cloud SQL
- Bigtable
- Cloud Storage
- Google ドライブ

ただし、データの整合性が取れないなどの制約はある。  
外部データソースにクエリする場合でも、 [外部データソースに対するテーブル定義ファイルの作成](https://cloud.google.com/bigquery/external-table-definition) は必要。

## 4. BigQuery へのデータの読み込み

[読み込み](https://cloud.google.com/bigquery/docs/loading-data) には以下の選択肢がある。

- バッチ読み込み
    - 単一のバッチオペレーションでソースデータ（CSV ファイル、外部データベース、ログファイルなど）を BigQuery テーブルに読み込む
    - 従来の抽出、変換、読み込み（ETL）ジョブはこのカテゴリに分類される
    - バッチ読み込みは、一回限りまたは定期的なスケジュールで行える
        - BigQuery Data Transfer Service の転送はスケジュールに基づいて実行
        - Cloud Composer などのオーケストレーション サービスを使用して、読み込みジョブをスケジュール
        - cron ジョブを使用して、スケジュールに従ってデータを読み込み
- ストリーミング
    - データを一度に 1 つずつ、または一括して送信し、ストリーミング API を直接呼び出すコードを記述できる
    - あるいは、Dataflow を Apache Beam SDK で使用してストリーミング パイプラインを設定することもできる
    - ストリーミングはデータサイズによって別途 **追加料金が発生** することに注意（ [参考](https://cloud.google.com/bigquery/pricing#data_ingestion_pricing) ）
- クエリ結果のテーブル追加・上書き
    - データ操作言語（DML）ステートメントを使用して、既存のテーブルへの一括挿入や、新しいテーブルでのクエリの結果の保存ができる
- サードパーティアプリケーションやサービスの利用
    - 一部のサードパーティ アプリケーションやサービスでは、データを BigQuery に取り込むためのコネクタが提供されている
    - [BigQuery Data Transfer Service](https://cloud.google.com/bigquery-transfer/docs/transfer-service-overview) を利用すれば Google Analytics などの Google のサービスや一部のクラウド/SaaS のストレージ/DWHからデータをインポートできる

なお、CSV ファイルなどからデータを読み込む際、形式などに不正がありインポートに失敗するレコードが含まれる場合、BigQueryの機能で自動でどうこうすることはできない。  
Dataflow などで行のバリデーションを実施した上でインポートするしかない。

## 5. BigQuery へのクエリ

### 5.1. クエリの種類

BigQuery には次のようなクエリ・ジョブがある。

- インタラクティブ クエリ
    - すぐに実行されるクエリ（デフォルト）
- バッチ クエリ
    - BigQuery はユーザーに代わって各バッチクエリをキューに格納し、アイドル状態のリソースが使用可能になると直ちに（通常は数分以内に）クエリを開始する
- クエリジョブ
    - データの読み込み、データのエクスポート、データのクエリ、データのコピーなど、BigQuery がユーザーに代わって実行するアクションのこと

### 5.2. クエリの結果

すべてのクエリ結果は **永続テーブル** または **一時テーブル** に保存される。

- 永続テーブル
    - ユーザーがアクセス可能なデータセットに含まれる新規または既存のテーブル
- 一時テーブル
    - 永続的なテーブルに書き込まれていない一時テーブルを使用してクエリ結果をキャッシュに保存する（通常 24 時間）
    - `CURRENT_TIMESTAMP()` や `NOW()` といった日時関数や `SESSION_USER()` 関数など、また、ワイルドカードを利用する場合はキャッシュは機能しない
    - また、 **ジョブ** 構成、GCPコンソール、従来のWeb UI、コマンドライン、またはAPIで宛先テーブルが指定されている場合もクエリ結果はキャッシュされない

## 6. テーブルスキーマの変更

[テーブルスキーマの変更](https://cloud.google.com/bigquery/docs/managing-table-schemas) を行う際、以下の場合その変更はネイティブにサポートされている。

- スキーマ定義への列の追加
- スキーマ定義からの列の削除またはドロップ
- 列のモードの REQUIRED から NULLABLE への緩和

しかし以下の場合、新しいテーブルを作成して、 SELECT-INSERT してデータ移行する必要がある。

- 列の名前の変更
- 列のデータ型の変更
- 列のモードの変更（REQUIRED 列を NULLABLE に緩和することを除く）

## 7. 承認済みのビュー

データセットに表示アクセス権を設定する場合、BigQuery では [承認済みのビュー](https://cloud.google.com/bigquery/docs/authorized-views) を作成する。  
承認済みビューを使用すると、元のソースデータへのアクセス権がないユーザーでも、クエリの結果を特定のユーザーやグループと共有できる。

## 7. コスト

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

また、 [カスタムコスト管理](https://cloud.google.com/bigquery/docs/custom-quotas#controlling_query_costs_using_bigquery_custom_quotas) を利用することにより、1 日に処理されるクエリデータの量に対する上限を指定することができる。