---
title: "Aurora"
date: 2021-01-30T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

<!--more-->

[Amazon Aurora](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html)についてまとめる。

- 1. [特徴](#1-特徴)
- 2. [設定](#2-設定)
- 3. [ストレージ](#3-ストレージ)

## 1. 特徴

- インスタンスとストレージが分離している
- ストレージがマルチ AZ HA クラスタ
- Protection Groups
  - ストレージは10GBごとに3AZにまたがって6つのコピーを持つ構成（この10GBの塊のことをProtection Group）
- BackTrack
  - データ巻き戻し機能
- クラスタキャッシュ管理による高速リカバリ
  - プライマリインスタンスが持っているキャッシュを、特定のレプリカと共有することができる機能
  - リードレプリカがプライマリ昇格した際の性能劣化を防ぐ
- 並列クエリ（Parallel Query）
  - インスタンス層がやっているクエリ実行などの処理をそのストレージ層のリソースにオフロードして、ストレージ側で処理してしまおうというのが並列クエリ
- Global Database
  - ストレージレイヤでのリージョン間レプリケーション
- Aurora Serverless
  - インスタンス層が需要に合わせて自動スケールする機能

## 2. 設定

Terraform リソースベースで記載する。

### 2.1. `aws_rds_cluster` と `aws_rds_cluster_instance`

Aurora はクラスタ内に複数インスタンス構築されると、ライターインスタンスとリーダーインスタンスに役割分担され、障害時はリーダーインスタンスがライターインスタンスに昇格する。  
なので、 `availability_zone` にてマルチ AZ 構成となるよう設定する必要がある。

`aws_rds_cluster` の設定は以下の通り。（どうでもいいもの・特殊なものは省略している）

- `apply_immediately` -> デフォルト・推奨：false
  - クラスタの変更を直ちに適用する(true) or 次回メンテナンスウィンドウで適用するか(false)
- `availability_zones` -> 設定不要
  - DB クラスタ内の DB クラスタインスタンスを作成する AZ のリスト
  - なお、 DB サブネットグループ（ `aws_db_subnet_group` ）の設定が反映されるため、明示的に設定する必要はない
- `backtrack_window` -> 要件がなければデフォルト：0
  - BackTrack（データ巻き戻し機能）の時間幅
  - 0 〜 259200 秒 (72 時間)
- `backup_retention_period` -> デフォルト：1
  - バックアップを保持する日数
- `cluster_identifier`
  - クラスタの識別子（nameの代わり）
- `copy_tags_to_snapshot`
  - クラスタに定義された `tags` をクラスタから作成したスナップショットにコピーするか否か
- `database_name`
  - データベース名
- `db_cluster_parameter_group_name`
  - クラスタ内の全ての DB インスタンスに適用されるパラメータグループ
- `db_instance_parameter_group_name` -> 利用しない
  - DB インスタンス単体に適用されるパラメータグループ
- `db_subnet_group_name`
  - DB インスタンスに関連づける DB サブネットグループ
- `deletion_protection` -> 推奨：true
  - DB インスタンスの削除保護を有効にするか否か
- `enable_http_endpoint` -> デフォルト・通常：false
  - HTTP エンドポイントを有効にするか否か
- `enabled_cloudwatch_logs_exports`
  - CloudWatch Logs にログ出力するログ種別を指定
  - `audit` 、 `error` 、 `general` 、 `slowquery` 、 `postgres` から指定（ PostgreSQL は `postgres` のみ）
- `engine_mode` -> 通常要件がない限り設定不要
  - エンジンモードの指定
  - `global` 、 `multimaster` 、 `parallelquery` 、 `provisioned` 、 `serverless` から指定
- `engine`
  - クラスタで利用するデータベースエンジン名
- `engine_version`
  - クラスタで利用するデータベースエンジンのバージョン
- `final_snapshot_identifier`
  - クラスタを削除する際に作成するスナップショット名
- `iam_database_authentication_enabled` -> デフォルト・通常：false
  - DB アクセス認証をID・パスワード認証ではなく、 IAM 認証にするか否か
- `iam_roles` -> 通常不要
  - クラスタに関連づける IAM ロールの ARN リスト
- `kms_key_id`
  - `storage_encrypted` を `true` にした際に利用する KMS キー
- `master_password`
  - マスターDBユーザパスワード
- `master_username`
  - マスターDBユーザ名
- `port`
  - DB ポート
- `preferred_backup_window`
  - バックアップウィンドウの時間指定
  - `"15:00-16:00"` みたいな感じで指定（UTC）
- `preferred_maintenance_window`
  - メンテナンスウィンドウの時間指定
  - `"sun:16:00-sun:17:00"` みたいな感じで指定（UTC）
- `replication_source_identifier` -> 通常設定不要
  - リードレプリカのソース DB として使用する DB クラスタまたはインスタンスの ARN を指定
- `skip_final_snapshot` -> 推奨：true
  - DB クラスタを削除する際に DB スナップショットを作成するか否か
- `snapshot_identifier`
  - DB クラスタをスナップショットから作成する場合に指定するスナップショットの ARN
- `source_region` -> 通常設定不要
  - 暗号化レプリカ DB クラスタの存在するリージョンを指定
- `storage_encrypted` true or false
  - DB クラスタを暗号化するか否か
- `vpc_security_group_ids`
  - DB クラスタに関連づける セキュリティグループの ID リスト

`aws_rds_cluster_instance` の設定は以下の通り。（どうでもいいもの・特殊なものは省略している）

- `identifier`
  - DB インスタンスの識別子
- `cluster_identifier`
  - DB インスタンスが所属する DB クラスタの識別子
- `engine`
  - インスタンスで利用するデータベースエンジン名
- `engine_version`
  - インスタンスで利用するデータベースエンジンのバージョン
- `instance_class`
  - インスタンスクラス
- `db_subnet_group_name`
  - DB インスタンスに関連づける DB サブネットグループ
- db_parameter_group_name
  - DB インスタンス単体に適用されるパラメータグループ
- `apply_immediately` -> デフォルト・推奨：false
  - データベースの変更を直ちに適用する(true) or 次回メンテナンスウィンドウで適用するか(false)
- `monitoring_role_arn`
  - 拡張モニタリングメトリクスを CloudWatch Logs へ送信する IAM ロールの ARN
- `monitoring_interval` -> デフォルト：0
  - 拡張モニタリングメトリクスの収集間隔
  - 0 、　1 、 5 、 10 、 15 、 30 、 60 から選択
- `promotion_tier` -> デフォルト：0
  - インスタンスレベルのフェイルオーバーの優先度を指定する項目
  - tier が低いリーダーは、ライターに昇格する優先度が高くなる
- `availability_zone`
  - DB クラスタインスタンスを作成する AZ
- `preferred_backup_window`
  - バックアップウィンドウの時間指定
  - `"15:00-16:00"` みたいな感じで指定（UTC）
- `preferred_maintenance_window`
  - メンテナンスウィンドウの時間指定
  - `"sun:16:00-sun:17:00"` みたいな感じで指定（UTC）
- `auto_minor_version_upgrade` -> デフォルト・推奨：true
  - メンテナンスウィンドウでマイナーエンジンアップグレードを行うか否か
- `performance_insights_enabled`
  - Performance Insights を有効にするか否か
- `performance_insights_kms_key_id`
  - `performance_insights_enabled` を `true` にした際に利用する KMS キー
- `performance_insights_retention_period` -> デフォルト：7
  - `performance_insights_enabled` を `true` にした際に利用する Performance Insights データの保持期間
  - 7 〜 731 日
- `copy_tags_to_snapshot`
  - インスタンスに定義された `tags` をインスタンスから作成したスナップショットにコピーするか否か

## 3. ストレージ

Aurora のストレージには以下の2種類がある。

1. ローカルストレージ
2. クラスタボリューム

### 3.1. ローカルストレージ

EC2 でいうインスタンスストアに相当し、DB インスタンスクラスに応じてそのサイズは固定値となっている。「一時ストレージ（Temporary storage）」と記載される場合もある。  
例えば Aurora MySQL においては、エラーログ、一般ログ、スロークエリログ、監査ログ、や InnoDB 以外の一時テーブルの保存に使用される。

### 3.2. クラスタボリューム

永続的にデータを保存するためのストレージで、このストレージは自動的に増加するのが Aurora の特徴。

### 3.3. メトリクス

- `FreeLocalStorage`
  - ローカルストレージの空き容量を示すメトリクス
  - ストレージ拡張が必要な場合はインスタンスクラスのスケールアップで対応
- `VolumeBytesUsed`
  - クラスタボリュームの容量を示すメトリクス
  - 10 GB 単位で自動拡張する
  - DROP TABLE、DROP DATABASE、TRUNCATETABLE などの SQL ステートメントを実行した場合に billed space つまり VolumeBytesUsed も減少する（Dynamic Resizing）
- `AuroraVolumeBytesLeftTotal`
  - クラスタボリュームの確保可能容量の残量を示すメトリクス
  - 最大 128 TiB までクラスタボリュームが自動的に増加するため、例えば 1 TiB の容量を（VolumeBytesUsed で）確保している場合、127 TiB となる