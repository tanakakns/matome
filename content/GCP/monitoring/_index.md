---
title: "監視"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

Cloud Logging、Cloud Monitoring、Cloud Trace、Debugger、Error Reporting

<!--more-->

1. [コンセプト](#1-コンセプト)
2. [Cloud Logging](#2-cloud-logging)
3. [Cloud Monitoring](#3-cloud-monitoring)

## 1. コンセプト

[オペレーションスイート](https://cloud.google.com/products/operations?hl=ja)（旧称 Stackdriver）

1. [Cloud Logging](https://cloud.google.com/logging/docs/concepts?hl=ja)
    - リアルタイムでログを収集・分析する大規模なフルマネージドサービス
        - フィルターや検索でログ分析でき、トラブルシューティングが加速
        - ログに基づいた指標（メトリクス）定義
        - VM インスタンスに対しては[Logging エージェント](https://cloud.google.com/logging/docs/agent?hl=ja)を導入する必要がある（ GKE ノードはデフォルトで導入済）
        - ログは **400 日間** 保存されるが延長することはできないので、長期保存するには Cloud Storage などへエキスポートする必要がある
    - アプリケーションとシステムのログデータのほか、GKE 環境 や Google Cloud サービスからのカスタム ログデータも取り込める
    - BigQuery、Google Cloud Storage、Pub/Subへのエクスポートできる
2. [Cloud Monitoring](https://cloud.google.com/monitoring/docs/concepts?hl=ja)
    - 組み込みの大規模なリソース指標（メトリクス）オブザーバビリティ
    - クラウドで実行されるアプリケーションのパフォーマンスや稼働時間、全体的な動作状況を確認できる
    - メトリクス・イベント・メタデータを収集し、**ワークスペース** を作成することでチャートやダッシュボードで可視化してアラート管理できる
    - AWS にも対応
3. [Cloud Trace](https://cloud.google.com/trace/docs/overview)
    - 各プログラミング言語に対応したライブラリを使用することで、HTTP ヘッダを挿入し、VM、コンテナー、AppEngine、トレースをキャプチャする
        - ロードバランサにも対応
    - URL ごとの統計、レイテンシの分布
4. Debugger
    - 本番環境でのコードのデバッグ（途中で止める）
5. Error Reporting
    - エラーをレポート、通知

- エージェント
    - インスタンスから情報を収集する VM に [Monitoring エージェント](https://cloud.google.com/monitoring/agent) と [Logging エージェント](https://cloud.google.com/logging/docs/agent?hl=ja) をインストールすることで収集可能となる

## 2. Cloud Logging

### 2.1. アクセス制御

IAM を使用して Google Cloud リソースのロギングデータへの [アクセス制御](https://cloud.google.com/logging/docs/access-control?hl=ja#permissions_and_roles) を実施できる。  

- `roles/logging.viewer`（ログビューア）（＝ `roles/viewer` ）
    - アクセスの透明性ログとデータアクセス監査ログ以外のすべての Logging 機能に対する読み取り専用権限
- `roles/logging.privateLogViewer`（プライベート ログビューア）
    - `roles/logging.viewer` に加え、アクセスの透明性ログとデータアクセス監査ログの読み取り権限
    - このロールは、_Required バケットと _Default バケットにのみ適用される
- `roles/logging.logWriter`（ログ書き込み）
    - サービスアカウントに付与して、ログの書き込みに十分な権限のみをアプリケーションに付与できる
    - ただし、閲覧権限はない
- `roles/logging.bucketWriter`（ログバケット書き込み）
    - サービス アカウントに付与すると、ログバケットにログを書き込むための十分な権限を Cloud Logging に付与できる
    - このロールを特定のバケットに制限するには、IAM 条件を使用する
- `roles/logging.configWriter`（ログ構成書き込み）は
    - ログベースの指標、除外、バケット、ビューを作成し、シンクをエクスポートする権限
    - これらのアクションに Logs Explorer（コンソール）を使用するには、 `roles/logging.viewer` を追加する
- `roles/logging.admin`（Logging 管理者）
    - Logging に関連するすべての権限
- `roles/logging.viewAccessor`（ログ表示アクセス者）
    - ビュー内のログを読み取るための権限
    - このロールを特定のバケット内のビューに制限するには、IAM 条件を使用
- `roles/editor`（プロジェクト編集者）
    - roles/logging.viewer の権限、ログエントリの書き込み権限、ログの削除権限、ログベースの指標の作成権限が含まれる
    - この役割では、エクスポート シンクの作成や、アクセスの透明性ログまたはデータアクセス監査ログの読み取りを行うことはできない
- `roles/owner`（プロジェクト オーナー）
    - Logging に対する完全アクセス権（アクセスの透明性ログとデータアクセス監査ログ）

### 2.1. ログの種類

- Google Cloud Platform のログ： GCP サービス固定ログ
- ユーザー作成のログ：ユーザ独自ログで、Logging エージェント、Cloud Logging API、または Cloud Logging クライアント ライブラリから取り込んだもの
- セキュリティログ
    - 監査ログ（[Cloud Audit Logs](https://cloud.google.com/logging/docs/audit)）
    - [アクセスの透過ログ](https://cloud.google.com/logging/docs/audit/access-transparency-overview)： Google スタッフによる操作ログ
- マルチクラウドとハイブリッド クラウドのログ： Azure や AWS など別クラウドから取り込んだログ

#### 2.1.1. 監査ログ

権限管理された上で、オペレーションを監視するログに以下の種類がある。（ [Cloud Audit Logs](https://cloud.google.com/logging/docs/audit) ）

- 管理アクティビティ監査ログ
    - リソース構成やメタデータ変更オペレーションが記録される
    - デフォルトで有効で無効にできない
- データアクセス監査ログ
    - リソース構成やメタデータ読み取り API 操作が記録される
    - デフォルトは無効
- システムイベント監査ログ
    - Google システムによって生成される管理アクション
- ポリシー拒否監査ログ
    - セキュリティ ポリシー違反のため Google Cloud サービスがユーザーまたはサービス アカウントへのアクセスを拒否したときに、ポリシー拒否監査ログを記録する

現状、アクティビティログというサービスがあるが、 Deprecated なので監査ログを使用すること。

### 2.2. エキスポート・シンク

Cloud Logging は以下のエキスポート・シンクをサポートする。  
gcloud 、Cloud Console、Google Cloud APIs ログシンクでログシンクのエクスポート先を作成してから、[集約シンク](https://cloud.google.com/logging/docs/export/aggregated_sinks) を作成する。

- Cloud Storage
    - Cloud Storage バケットに保存される JSON ファイル
- BigQuery
    - BigQuery データセットに作成されるテーブ。
- Pub/Sub
    - Pub/Sub トピックに配信される JSON メッセージ
    - サードパーティによる Logging との統合サービス（Splunk など）をサポート

## 3. Cloud Monitoring

- [ワークスペース](https://cloud.google.com/monitoring/workspaces?hl=ja#accounts) は、1 つ以上の Google Cloud プロジェクトまたは AWS アカウントに含まれるリソースをモニタリングするためのツール
    - 各ワークスペースには、Google Cloud プロジェクトや AWS アカウントなど、1～100 個のモニタリング対象プロジェクトを作成できる
    - ワークスペースの数に制限はないが、Google CloudプロジェクトとAWSアカウントを複数のワークスペースでモニタリングすることはできない
- **ホストプロジェクト**
    - すべてのワークスペースには、ホストプロジェクトがある
    - ワークスペースの作成に使用した Google Cloud プロジェクトが、ワークスペースのホスト プロジェクト
- モニタリング対象プロジェクト
    - ホストプロジェクトでワークスペースを作成すると、Google Cloud プロジェクトと AWS アカウントが追加できるようになる
    - 新しい空白の Google Cloud プロジェクトを使用してワークスペースをホストしてから、モニタリングするプロジェクトと AWS アカウントをワークスペースに追加するのがよい
        - ホスト プロジェクトと Workspace の名前を自由に選択でき、ワークスペース間でモニタリング対象プロジェクトを柔軟に移動できる
- [カスタムメトリクス](https://cloud.google.com/monitoring/custom-metrics)

### 3.1. アラート

[アラートの動作](https://cloud.google.com/monitoring/alerts/concepts-indepth)