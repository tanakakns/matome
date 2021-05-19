---
title: "ビッグデータ"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 9
---

<!--more-->

|サービス|モデル|形式|ユースケース|対応AWS|
|:---|:---|:---|:---|:---|
|[Cloud Pub/Sub](./pubsub)|||||

## データ分析パイプライン

「取り込み」 -> 「処理」 -> 「分析」 -> 「可視化」の順で処理をする。

- 取り込み
    - Cloud Pub/Sub：ストリーミングの場合はこちら
    - Cloud Storage：ストリーミングではなく一時保存する場合はこちらでも
- 処理
    - Cloud Dataflow（推奨）： Apache Beam 準拠の ETL 処理、サーバレス
    - Cloud Dataproc：マネージド Hadoop/Spark、クラスタ管理が必要、既存の Hadoop/Sparkジョブがある場合はこちらでも
- 分析
    - BigQuery
- 可視化
    - DataPortal

## Dataflow と Dataproc

- Google Cloud Dataflow
    - Dataflowは、サーバーレス、高速、費用効果の高い統合ストリームおよびバッチデータ処理
    - バッチパイプラインとストリーミングパイプラインの両方に適する
    - 参照：https://cloud.google.com/dataflow
- Google Cloud Dataproc
    - オープンソースのデータと分析の処理をクラウドで高速、簡単、かつ安全にする
    - フルマネージドのApache Spark、Apache Hadoop、Presto、およびその他のOSSクラスターを構築できる
    - しかし、Dataprocは、バッチおよびストリーム処理には適していない
    - 参照：https://cloud.google.com/dataproc#section-5

## その他ビッグデータ系サービス

- Cloud Data Loss Prevention
    - GCP内で処理されるデータを監査し、機密データ(電話番号、メールアドレス等の個人情報等）の有無の検知やマスキング処理等が可能なサービス
- [Dataprep by Trifacta](https://cloud.google.com/dataprep/)
    - 分析用に異種の大規模なデータセット（構造化データと非構造化データ）の視覚的な探索、データクレンジング、構造化されたデータに変換して準備を行うためのインテリジェントデータサービス
    - Cloud Storage・BigQuery・デスクトップに保存されたデータを処理して、データを洗練してからCloud Storage・BigQueryにエクスポートできる
    - Qwiklabs の [Cloud Dataprep でデータ変換パイプラインを作成する](https://googlecourses.qwiklabs.com/focuses/11580?parent=catalog) で体験できる。
- [Cloud Datalab](https://cloud.google.com/datalab)
    - データを探索、分析、変換、視覚化し、GCPで機械学習モデルを構築するために作成された強力なインタラクティブツール
    - Compute Engineで実行され、複数のクラウドサービスに簡単に接続できるため、データサイエンスのタスクに集中できる
    - 一方でデータのクレンジングには適さない
- [Cloud Composer](https://cloud.google.com/composer)
    - Apache Airflow で構築された、フルマネージドのワークフローオーケストレーションサービス

## Dataprep by Trifacta

ここでは、 Dataprep by Trifacta を利用して以下を実施する。

- BigQuery データセットを Cloud Dataprep に接続する。
- Cloud Dataprep でデータセットの品質を調べる。
- Cloud Dataprep でデータ変換パイプラインを作成する。
- BigQuery に出力される変換ジョブをスケジュールする。