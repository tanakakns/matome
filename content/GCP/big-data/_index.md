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
    - Chrome ブラウザでのみ操作可能
    - Qwiklabs の [Cloud Dataprep でデータ変換パイプラインを作成する](https://googlecourses.qwiklabs.com/focuses/11580?parent=catalog) で体験できる。
- [Cloud Datalab](https://cloud.google.com/datalab)
    - データを探索、分析、変換、視覚化し、GCPで機械学習モデルを構築するために作成された強力なインタラクティブツール
        - データクレンジングには適していない
    - Compute Engineで実行され、複数のクラウドサービスに簡単に接続できるため、データサイエンスのタスクに集中できる
    - 一方でデータのクレンジングには適さない
- [Cloud Composer](https://cloud.google.com/composer)
    - Apache Airflow で構築された、フルマネージドのワークフローオーケストレーションサービス

## Dataprep by Trifacta

Cloud Dataprep は Chrome ブラウザからのみ利用できる。  
Cloud Console から以下の手順で開く。

 - ナビゲーション メニューを開き、[Dataprep] を選択 -> 利用規約に同意 -> いろいろ許可
 - [Sign in with Google] ウィンドウが表示された場合は GCP のアカウントでログイン -> 規約などに同意
 - ストレージ バケットのデフォルトの場所を使用するように求められたら [Continue] をクリック

 ## ユースケース

 ### Google Cloud での IoT 分析パイプラインの構築

 要件は以下。

- Cloud IoT Core を使用して MQTT ベースのデバイスを接続、管理します（シミュレートされたデバイスを使用します）。
- Cloud Pub/Sub を使用して Cloud IoT Core から情報のストリームを取り込みます。
- Cloud Dataflow を使用して IoT データを処理します。
- BigQuery を使用して IoT データを分析します。

手順は以下。  
まずは、 API を有効にする。

- ナビゲーション メニュー > [API とサービス] をクリックし、以下の API を有効化する
    - Cloud IoT API
    - Cloud Pub/Sub API
    - Dataflow API

Cloud Pub/Sub トピックの作成する。

- ナビゲーション メニュー > [Pub/Sub] > [トピック] へ移動し、 `iotlab` トピックを作成
- 作成したトピックの [View permissions] を選択し、[権限] ダイアログで、[ADD MEMBER] をクリックし、[新しいメンバー] に `cloud-iot@system.gserviceaccount.com` を追加
- [Select a role] メニューから新しいメンバーに [Pub/Sub] > [Pub/Sub パブリッシャー] 役割を付与して「保存」

Pub/Sub に送信されたメッセージを集約し、BigQuery に保存するため、BigQuery データセットを作成する。

- ナビゲーション メニュー > [BigQuery] に移動、[完了] をクリック
- 左側のメニューからプロジェクト ID をクリック ->  [データセットを作成] をクリック -> データセットに「iotlabdataset」という名前を付け、他のすべてのフィールドはそのままにして、[データセットを作成] 
- データセットをクリックして、コンソールの右側にある [+ テーブルを作成] をクリック
- ソース フィールドが [空のテーブル] に設定されていることを確認
- [送信先] セクションの [テーブル名] フィールドに、「sensordata」と入力
- [スキーマ] セクションで、[+ フィールドを追加] ボタンをクリックして次のフィールドを追加
    - フィールド名として「timestamp」と入力し、フィールドの [型] を [TIMESTAMP] に設定
    - フィールド名として「device」と入力し、フィールドの [型] を [STRING] に設定
    - フィールド名として「temperature」と入力し、フィールドの [型] を [FLOAT] に設定
- 他のデフォルト設定は変更せずに、[テーブルを作成] をクリック

Cloud Dataflow パイプラインの作業スペースとして Cloud Storage バケットを作成する。

- ナビゲーション メニュー > [Storage] に移動
- [バケットを作成] をクリックし、[バケットに名前を付ける] には「qwiklabs-gcp-03-b799ccdd7f75-bucket」
- [デフォルトのストレージ クラスを選択する] で、まだ選択されていない場合は [Multi-Regional] をクリック
- [場所] で最も近いもの（ asia ）を選択し、[作成] をクリック

Cloud Dataflow パイプラインを設定する。  
ここでは、Pub/Sub からセンサーデータを読み取り、一定の期間の最高気温を計算し、これを BigQuery に書き込むストリーミング データ パイプラインを設定する。

- ナビゲーション メニュー > [Dataflow] に移動
- 上部のメニューバーの [テンプレートからジョブを作成] をクリック
- ジョブ作成ダイアログの [ジョブ名] に「iotlabflow」と入力
- [リージョン エンドポイント] では、リージョンに [us-central1] を選択
- [Dataflow テンプレート] には、[Pub/Sub Topic to BigQuery] を選択
    - このテンプレートを選択すると、フォームが更新され、下に新しいフィールドが表示される
- [Pub/Sub input topic] には、`projects/qwiklabs-gcp-03-b799ccdd7f75/topics/iotlab`
- [BigQuery output table] には、 `qwiklabs-gcp-03-b799ccdd7f75:iotlabdataset.sensordata`
- [一時的なロケーション] には、`gs://qwiklabs-gcp-03-b799ccdd7f75-bucket/tmp/`
- [オプション パラメータ] をクリックして以下を設定
    - [最大ワーカー数] に「2」と入力
    - [マシンタイプ] に「n1-standard-1」と入力
- [ジョブを実行] をクリック

Compute Engine VM を準備する。  
事前にプロビジョニング済みの iot-device-simulator という VM インスタンスを使用して、MQTT 接続された IoT デバイスをエミュレートする Python スクリプトのインスタンスを実行する。  
また、デバイスをエミュレートする前に、この VM インスタンスを使用して Cloud IoT Core デバイス レジストリにデータを入力する。

- ナビゲーション メニュー > [Compute Engine] > [VM インスタンス] に移動し、 `iot-device-simulator` の[SSH] プルダウン矢印をクリックして、[ブラウザ ウィンドウで開く] を選択
- 以下を実行

```bash
$ sudo pip install virtualenv
$ virtualenv -p python3 venv
$ source venv/bin/activate
$ sudo apt-get remove google-cloud-sdk -y
$ curl https://sdk.cloud.google.com | bash
$ exit

$ source venv/bin/activate
$ gcloud init
# [1] 982998799090-compute@developer.gserviceaccount.com
# [2] Log in with a new account
# と出た場合は 2 を選択
https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=32555940559.apps.googleusercontent.com&r
edirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&scope=openid+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.ema
il+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.adm
in+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcompute+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Faccounts.reauth&stat
e=AQXJKoqhVh09dIRVOaK1xeMqEeZQ1l&access_type=offline&code_challenge_method=S256&prompt=consent&code_challenge=356k-
gg6lJMSpeXG7G6X3Ey8l_YzI_Sdiqkhpc23Bco
```

- 表示された URL をクリックすると、新しいブラウザ ウィンドウが開き、確認コードが表示される（ログインを求められた場合はログインする）
    - 4/1AY0e-g7q-H3BMTQfsIZ3k79YIs6016W_NSH0wsQQsO-0FNzK8a_PS62Jw5U
- [Enter verification code:] プロンプトに対して確認コードを貼り付けて、Enter キーを押す
- [Pick cloud project to use] へのレスポンスで、Qwiklabs によって作成されたプロジェクトを選択
- 次のコマンド（ `gcloud components update` ）を入力して、SDK のコンポーネントが最新であることを確認
- 次のコマンド（ `gcloud components install beta` ）を入力して、ベータ コンポーネントをインストール
- 次のコマンド（ `sudo apt-get update` ）を入力して、Debian Linux パッケージ リポジトリに関するシステムの情報を更新
- 次のコマンド（ `sudo apt-get install python-pip openssl git -y` ）を入力して、必要な各種ソフトウェア パッケージがインストールされていることを確認
- pip を使用して必要な Python コンポーネントを追加（ `pip install pyjwt paho-mqtt cryptography` ）
- 次のコマンド（ `git clone http://github.com/GoogleCloudPlatform/training-data-analyst` ）を入力して、このラボで分析するデータを追加

IoT デバイスのレジストリを作成する。  
iot-device-simulator VM インスタンスの SSH セッションで、次のコマンドを実行する。

```bash
$ export PROJECT_ID=qwiklabs-gcp-03-b799ccdd7f75
$ export MY_REGION=asia-east1
$ gcloud beta iot registries create iotlab-registry \
   --project=$PROJECT_ID \
   --region=$MY_REGION \
   --event-notification-config=topic=projects/$PROJECT_ID/topics/iotlab
```

暗号鍵ペアを作成する。  
IoT デバイスが Cloud IoT Core に安全に接続できるようにするには、暗号鍵ペアを作成する必要がある。  
iot-device-simulator VM インスタンスの SSH セッションで、次のコマンドを入力して適切なディレクトリに鍵ペアを作成。

```bash
$ cd $HOME/training-data-analyst/quests/iotlab/
$ openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem \
    -nodes -out rsa_cert.pem -subj "/CN=unused"
```

この openssl コマンドにより、RSA 暗号鍵ペアが作成され、rsa_private.pem というファイルに書き込まれる。  
次にシミュレートされたデバイスをレジストリに追加する。  
iot-device-simulator VM インスタンスの SSH セッションで、次のコマンドを入力して temp-sensor-buenos-aires というデバイスを作成する。

```bash
$ gcloud beta iot devices create temp-sensor-buenos-aires \
  --project=$PROJECT_ID \
  --region=$MY_REGION \
  --registry=iotlab-registry \
  --public-key path=rsa_cert.pem,type=rs256
$ gcloud beta iot devices create temp-sensor-istanbul \
  --project=$PROJECT_ID \
  --region=$MY_REGION \
  --registry=iotlab-registry \
  --public-key path=rsa_cert.pem,type=rs256
```

シミュレートされたデバイスを実行する。  
iot-device-simulator VM インスタンスの SSH セッションで、以下のコマンドを入力して、pki.google.com から CA ルート証明書を適切なディレクトリにダウンロード。

```bash
$ cd $HOME/training-data-analyst/quests/iotlab/
$ wget https://pki.google.com/roots.pem
```

以下のコマンドを入力して、シミュレートされた 1 つ目のデバイスを実行。

```bash
$ python cloudiot_mqtt_example_json.py \
   --project_id=$PROJECT_ID \
   --cloud_region=$MY_REGION \
   --registry_id=iotlab-registry \
   --device_id=temp-sensor-buenos-aires \
   --private_key_file=rsa_private.pem \
   --message_type=event \
   --algorithm=RS256 > buenos-aires-log.txt 2>&1 &
```

以下のコマンドを入力して、シミュレートされた 2 つ目のデバイスを実行。

```bash
$ python cloudiot_mqtt_example_json.py \
   --project_id=$PROJECT_ID \
   --cloud_region=$MY_REGION \
   --registry_id=iotlab-registry \
   --device_id=temp-sensor-istanbul \
   --private_key_file=rsa_private.pem \
   --message_type=event \
   --algorithm=RS256
```

テレメトリー データは、シミュレートされたデバイスから Cloud IoT Core を経由して Cloud Pub/Sub トピックに流れる。  
データフロー ジョブは Pub/Sub トピックからメッセージを読み取り、その内容を BigQuery テーブルに書き込む。

次に、BigQuery を使用してセンサーデータを分析する。

- Cloud Console でナビゲーション メニューを開き、[BigQuery] を選択
- クエリエディタに次のクエリを入力し、[実行] をクリック

```sql
SELECT timestamp, device, temperature from iotlabdataset.sensordata
ORDER BY timestamp DESC
LIMIT 100
```

### Dataflow と BigQuery を使用した Google Cloud での ETL 処理

