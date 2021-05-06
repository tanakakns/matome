---
title: "Cloud Functions"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 5
---

<!--more-->

[Cloud Functions](https://cloud.google.com/functions/docs/how-to?hl=ja)

## [イベントとトリガー](https://cloud.google.com/functions/docs/concepts/events-triggers)

イベントとは、データベースでのデータの変更、ストレージ システムへのファイルの追加、新しい仮想マシン インスタンスの作成など。  
イベントは対応の有無にかかわらず発生する。イベントへの対応（レスポンス）は、トリガーを作成して行う。  
トリガーは、いわば特定のイベントや一連のイベントに関心があることの宣言であり、関数をトリガーにバインドすると、イベント情報を取得してなんらかの対応をとることができる。

## ユースケース

|ユースケース|説明|
|:---|:---|
|データ処理 / ETL|ファイルの作成、変更、削除などの Cloud Storage イベントをリッスン。画像処理、動画のコード変換、データの検証と変換、インターネット上の任意のサービスの呼び出しなどといった操作。|
|Webhook|シンプルな HTTP トリガーによって、GitHub、Slack、Stripe などのサードパーティ システムをはじめ、HTTP リクエストを送信できるあらゆる場所で発生したイベントに応答。|
|軽量 API|軽量な疎結合のロジックを組み合わせることで、アプリケーションのビルドや拡張を迅速に実行。関数は、イベントでトリガーされるようにすることも、HTTP(S) で直接呼び出すことも。|
|モバイル バックエンド|アプリ デベロッパー向けに Google が提供しているモバイル プラットフォームの Firebase を使用すればモバイル バックエンドを記述できる。これにより、Firebase Analytics、Firebase Realtime Database、Firebase Authentication、Firebase Storage からのイベントをリッスンして応答できる。|
|IoT|デバイスの膨大なストリーミング データを Cloud Pub/Sub に取り込み、Cloud Functions を呼び出してそのデータを処理、変換、保存するといった操作を完全にサーバーレスな形で実現。|

## ユースケース

Cloud Pub/Sub トピックに関する Cloud Functions のイベントを扱う。（ [Google Cloud Pub/Sub: Google 規模のメッセージ サービス](https://cloud.google.com/pubsub/architecture) ）  
イベント パラメータとコールバック パラメータについて詳しくは、 [バックグラウンド関数](https://cloud.google.com/functions/docs/writing/background) を確認。

```bash
# 関数コード用のディレクトリを作成
$ mkdir gcf_hello_world
$ cd gcf_hello_world

# index.js を作成し、編集
/**
* Pub/Sub でトリガーされるバックグラウンド Cloud Function 関数。
* この関数は index.js によってエクスポートされ、トリガー トピックが
* メッセージを受信すると実行されます。
*
* @param {object} data The event payload.
* @param {object} context The event metadata.
*/
exports.helloWorld = (data, context) => {
const pubSubMessage = data;
const name = pubSubMessage.data
    ? Buffer.from(pubSubMessage.data, 'base64').toString() : "Hello World";

console.log(`My Cloud Function: ${name}`);
};
```

```bash
# Cloud Storage バケット作成
$ gsutil mb -p [PROJECT_ID] gs://[BUCKET_NAME]
# gsutil mb -p qwiklabs-gcp-02-5fcdb48e7b6e gs://qwiklab-sample1234
```

次に新しい関数をデプロイするのだが、 `--trigger-topic` 、 `--trigger-bucket` 、 `--trigger-http` のいずれかを指定する必要がある。  
ここでは、 `--trigger-topic` に `hello_world` を指定する。

```bash
$ gcloud functions deploy helloWorld \
  --stage-bucket qwiklab-sample1234 \
  --trigger-topic hello_world \
  --runtime nodejs8

# ステータスの確認
$ gcloud functions describe helloWorld
```

関数をデプロイしてそれがアクティブになっていることを確認したら、関数がイベントを検出してメッセージをクラウド ログに書き込むかどうかをテストする。

```bash
# 関数のメッセージテストを実行
$ DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
executionId: u4h75ec5oy6m

# ログを確認
$ gcloud functions logs read helloWorld
LEVEL  NAME        EXECUTION_ID  TIME_UTC                 LOG
D      helloWorld  u4h75ec5oy6m  2021-05-06 12:37:28.571  Function execution took 219 ms, finished with status: 'ok'
I      helloWorld  u4h75ec5oy6m  2021-05-06 12:37:28.546  My Cloud Function: Hello World!
D      helloWorld  u4h75ec5oy6m  2021-05-06 12:37:28.352  Function execution started
```