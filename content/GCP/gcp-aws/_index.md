---
title: "GCP と AWS の比較"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

<!--more-->

## 違い

[AWS/Azure/GCPサービス比較 2021.04](https://qiita.com/hayao_k/items/906ac1fba9e239e08ae8)

- IAM
    - Cloud IAMはGoogleアカウントに対してプロジェクト毎にロールを割り当てる。（AWSのように管理者が独自のIAMアカウントを作って管理するわけではない）
    - 階層になっているIAMだが、階層の下に対してポリシーを継承できるものの下でより強い権限にてポリシーを上書きすることができる
- プロジェクトの概念
    - AWSでは契約ごとに払い出されるアカウント払い出されるが、GCPではプロジェクトという単位になっており、いくつでも自由に作れる
    - プロジェクト単位で課金やIAMを分けられる。
- ネットワーク
    - GCPではVPCを作ると初めからマルチリージョンで作られる / ゾーン(AZ相当)を跨ったサブネットが作れる
    - 外部IPアドレス（EIP相当）は、テンポラリーIPを昇格させることができる。また、グローバルに負荷分散できるIPも取得できる。
- データベース
    - CloudSQL（RDS相当）にOracleはない（Bare Metal Solution にある） 
    - CloudSQL に SQL Server はある、、、
- API
    - 各サービスを操作するAPIがあるのだが初期で有効になっているものはほとんどなく、ユーザが意図的に有効かしないとつかえない
- ストレージ
    - AWSのGlacierはデータを読み出す際に数時間かかるが、GCPのクラウドストレージのオプションであるColdLineやNearLineは同じようなサービスでありながら、通常のストレージと同じ速度で読み出しができる
- LoadBalancer
    - グローバル向けに HTTPロードバランサとSSLロードバランサとTCPロードバランサ、リージョン向けにネットワークロードバランサ（TCP/UDP/SSL) がある
    - 内部向けにはインターナルロードバランサがある（HTTP(S)/TCP/UDP）
    - グローバル向けの方はリバースプロキシサーバで、リージョン向け/内部向けはロードバランサ
    - GCPでグローバルIPを取得し、LBを前段におくとマルチリージョンでの負荷分散が行われる
- 監視
    - Stackdriver(CloudWatch相当)の有料のプレミアム階層では、AWSのリソースも同じダッシュボード上で監視ができる
    - Stackdriverアカウントに複数のGCPアカウントやAWSアカウントを紐付けることができる。
    - リソースを個々のインスタンスではなく、「Webサーバグループ」などとまとめて監視することもできる。

## GCP サービス一覧

[一行でわかる Google Cloud Platform](https://cloud.google.com/blog/ja/products/gcp/google-cloud-platform-paperprint)

### API プラットフォームとエコシステム

|サービス|一行説明|
|:---|:---|
|Apigee API Platform|API の開発ツール。開発からデプロイ、公開、モニタリング等を一元管理|
|Apigee Sense|悪意のある攻撃から API を保護|
|API Monetization|API のマネタイズツール|
|API Analytics|API のモニタリング及び分析|
|Cloud Endpoints|GCP 向けの API ゲートウェイ管理サービス|
|Cloud Launcher|GCP ユーザー向けに提供されているマーケットプレイス。サードパーティーが提供する製品を購入可能|

### Internet of Things（IoT）

|サービス|一行説明|
|:---|:---|
|Cloud IoT Core|IoT センサーやデバイスの管理・データ取込のためのマネージドサービス|

### データ転送

|サービス|一行説明|
|:---|:---|
|Google Transfer Appliance|専用機器を利用した大規模データ移行サービス|
|Cloud Storage Transfer Service|Google クラウドへのデータ移行サービス|
|BigQuery Data Transfer Service|自動的に一括で BigQuery にデータをロードするサービス|

### ID 管理とセキュリティ

|サービス|一行説明|
|:---|:---|
|Cloud IAM|GCP 各種サービスやバックエンドへのアクセス管理|
|Cloud Identity-Aware Proxy|ウェブサービスへの Google アカウントによる認証とアクセス制御|
|Data Loss Prevention API|クレジットカード番号等の機密情報を自動で検知、マスキング|
|Cloud Key Management Service|Google 以外が提供する暗号鍵を生成・管理・利用するためのサービス|
|Cloud Resource Manager|GCP 各種サービスやバックエンドへのアクセスを組織やグループの単位で管理|
|Cloud Security Scanner|App Engine で開発されたサービスを対象としたセキュリティスキャン。一般的な脆弱性を検出|
|Security Key Enforcement|2 段階認証を必須化するサービス|

### モニタリングツール

|サービス|一行説明|
|:---|:---|
|Stackdriver Monitoring|インフラストラクチャを含むシステム全体のモニタリング|
|Stackdriver Logging|インフラストラクチャを含むシステム全体のログ管理|
|Stackdriver Error Reporting|アプリケーションのエラー管理|
|Stackdriver Trace|アプリケーションのパフォーマンスに影響するボトルネックを検出・分析|
|Stackdriver Debugger|本番環境でのリアルタイムなデバッガー|
|Cloud Deployment Manager|システム全体の環境をテンプレートから自動的に展開|
|Cloud Console|GCP 管理用ウェブコンソール|
|Cloud Shell|ブラウザでアクセスするコマンド入力インターフェース|
|Cloud Console Mobile App|GCP 管理用モバイルアプリ（Android / iOS 対応）|
|Cloud Billing API|経理システム等から接続可能な請求情報を取得する API|
|Cloud APIs|GCP 管理用 API。GCP の管理をシステムやプログラムから行うための API 群|

### 開発者向けツール

|サービス|一行説明|
|:---|:---|
|Cloud SDK|GCP の管理・実行をシステムやプログラムから行うための SDK|
|Container Builder|アプリと実行環境をパッケージングし、コンテナにするサービス|
|Container Registry|コンテナの登録と管理|
|Cloud Source Repositories|ソースコード管理システム。プライベートで Git リポジトリをホスティングするサービス|
|Cloud Tools for Android Studio|Android Studio 上で GCP を利用するための支援ツール|
|Cloud Tools for IntelliJ|IntelliJ 上で GCP を利用するための支援ツール|
|Cloud Tools for PowerShell|PowerShell 上で GCP を利用するための支援ツール|
|Cloud Tools for Visual Studio|Visual Studio 上で GCP を利用するための支援ツール|
|Cloud Tools for Eclipse|Eclipse 上で GCP を利用するための支援ツール|
|Gradle App Engine Plugin|Gradle App Engine Plugin|
|Maven App Engine Plugin|Maven App Engine Plugin|

### ストレージ

|サービス|一行説明|
|:---|:---|
|Cloud Storage|クラウド上のストレージ。オブジェクトストレージ|
|Nearline|月 1 回程度のアクセスを想定したアーカイブストレージ|
|Coldline|年 1 回程度のアクセスを想定したアーカイブストレージ|
|Persistent Disk|仮想マシン（VM）用 ストレージ|

### コンピュート

|サービス|一行説明|
|:---|:---|
|Compute Engine|仮想マシンをはじめとする、仮想マシン用ストレージやネットワーク|
|Google App Engine|自動で柔軟にスケール可能なフルマネージドかつサーバレスのウェブアプリ向けバックエンド|
|Kubernetes Engine|コンテナを簡単に管理するための、Kubernetes マネージドサービス|
|Cloud Functions|ファイル追加等のイベント単位で処理・実行が可能なサーバーレス環境|

### 機械学習（Machine Learning）

|サービス|一行説明|
|:---|:---|
|Cloud Machine Learning Engine|機械学習のモデルをトレーニングするためのマネージドサービス|
|Cloud Job Discovery|機械学習を活用した求人情報に特化した検索用 API|
|Cloud Natural Language|自然言語処理用 API|
|Cloud Speech API|音声をテキストに変換する API|
|Cloud Text-to-Speech|テキストを音声に変換する API|
|Cloud Translation API|ニューラルネットを活用した機械翻訳（NMT）API|
|Cloud Vision API|画像認識用 API （自動のラベル付、文字抽出、顔検出）|
|Cloud Video Intelligence|動画に写り込んだ内容を自動的に検出及びラベル付、シーン分割する API|
|Cloud AutoML|高品質かつビジネスニーズに即した独自の機械学習モデルを構築可能|
|Dialogflow Enterprise Edition|様々なプラットフォームに対応する会話アプリ開発サービス|

### ビッグデータ

|サービス|一行説明|
|:---|:---|
|Google BigQuery|ビッグデータ用分析システム。超並列分散データウェアハウス|
|Cloud Dataflow|マネージド ストリーム / バッチデータ処理。IoT センサーから収拾したデータ等の前処理向けシステム|
|Cloud Dataproc|お手軽マネージド Spark / Hadoop サービス。IoT センサーから収拾したデータ等の前処理向けシステム|
|Cloud Dataprep|ブラウザで設定可能なストリーム / バッチデータ処理。IoT センサーから収拾したデータ等の前処理向けシステム|
|Cloud Composer|ワークフローを管理するための Apache Airflow マネージドサービス|
|Cloud Pub/Sub|メッセージキューサービス。グローバルかつリアルタイムに集まるデータを保管・処理|
|Google Data Studio|データを簡単に可視化するダッシュボード|
|Cloud Datalab|データをインタラクティブに可視化し分析するためのツール|

### ネットワーク

|サービス|一行説明|
|:---|:---|
|Virtual Private Cloud|Google が管理するネットワーク内に構築するプライベートな超大規模ネットワークサービス|
|Google Load Balancing|超大規模ネットワーク向け、グローバル負荷分散サービス。ロードバランサー|
|Cloud Armor|GCP 上に構築されたシステムやサービスを DDoS 攻撃等から保護|
|Cloud CDN|コンテンツ配信ネットワーク|
|Cloud DNS|高信頼の DNS サービス|
|Google Cloud VPN|VPN 経由による GCP への直接接続サービス|
|Direct Peering|GCP へのダイレクトピアリングによる直接接続サービス|
|Carrier Peering|GCP へのキャリアパートナー経由のピアリングによる直接接続サービス|
|Dedicated Interconnect|専用線で自社環境から GCP へ繋ぐ接続サービス|
|Partner Interconnect|サービスプロバイダー経由で自社環境から GCP へ繋ぐ接続サービス|

### モバイル（Firebase）

|サービス|一行説明|
|:---|:---|
|Realtime Database|マルチデバイスにリアルタイムに同期できるデータベース|
|Cloud Firestore|マルチデバイスにリアルタイムに同期でき、スケーラビリティが向上したデータベース|
|Cloud Storage|クラウド上のストレージ。オブジェクトストレージ|
|Hosting|ウェブコンテンツのホスティングサービス|
|Authentication|Google や Twitter 等のアカウントによるアプリ認証を簡単に追加できるサービス|
|Cloud Functions|ファイル追加等のイベント単位で処理・実行が可能なサーバーレス環境|
|Test Lab for Android|実際の Android 搭載デバイスでアプリを実行・テストするサービス|
|Performance Monitoring|アプリケーションのパフォーマンスモニタリング|
|Crashlytics|クラッシュレポーティングと分析|
|Cloud Messaging|デバイスへのメッセージ送信 デバイスへの通知送信|

### データベース

|サービス|一行説明|
|:---|:---|
|Cloud SQL|MySQL / PostgreSQL / SQL Server のマネージドサービス|
|Cloud Bigtable|HBase 互換の NoSQL DB|
|Cloud Datastore|水平スケールする NoSQL DB|
|Cloud Spanner|リレーショナル DB と NoSQL の特性を備えた DB。データをグローバルで複製・同期|
|Cloud Memorystore for Redis|Redis のインメモリー型データストア マネージドサービス|