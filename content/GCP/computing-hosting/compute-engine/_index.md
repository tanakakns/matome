---
title: "Compute Engine"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

<!--more-->

**Compute Engine** は Google のインフラストラクチャ上に仮想マシンを構築するサービス。

1. [コンセプト](#1-コンセプト)
2. [インスタンス作成](#2-インスタンス作成)
3. [ディスクの作成とアタッチ](#3-ディスクの作成とアタッチ)
4. [ファイアウォールルールの作成](#4-ファイアウォールルールの作成)
5. [インスタンスへの接続](#3-インスタンスへの接続)
6. [Cloud Monitoring](#4-cloud-monitoring)
7. サービスアカウント
8. コマンド整理

- [公式ガイド](https://cloud.google.com/compute/docs/how-to?hl=ja)
- [Compute Engine のリージョン選択に関するベスト プラクティス](https://cloud.google.com/solutions/best-practices-compute-engine-region-selection?hl=ja)

## 1. コンセプト

1. [VM インスタンス](#11-vm-インスタンス)
2. [マシンタイプ](#12-マシンタイプ)
3. [ストレージオプション](#13-ストレージオプション)
4. [イメージ](#14-イメージ)
5. [スナップショット](#15-スナップショット)
6. [ゲスト環境](#16-ゲスト環境)
7. [メタデータサーバー](#17-メタデータサーバー)
8. [ライブマイグレーション](#18-ライプマイグレーション)

### 1.1. VM インスタンス

マシンタイプとストレージからなる。  
VM オプションとして「可用性ポリシー」「サービスアカウント」「プリエンプティブル VM」「Shield VM」がある。

- 可用性ポリシー
    - メンテナンスイベントやクラッシュ発生時のインスタンスの動作を設定できる
    - メンテナンスイベント発生時の動作
        - 移行：ライブマイグレーションされる（推奨）
        - 終了：シャットダウン信号を受信して停止する
    - クラッシュ発生時の動作
        - 自動再起動オン：ユーザ操作意外でインスタンスが終了した場合に自動再起動する（推奨）
        - 自動再起動オフ：自動再起動しない
- サービスアカウント
    - インスタンスまたはアプリケーションが API リクエストをユーザの代わりに実行できるようにするための ID
    - デフォルトでは「プロジェクト編集者」の権限をもつサービスアカウントが付与されるため、適宜ユーザ定義したサービスアカウントを作成・付与する
- [プリエンプティブル VM](https://cloud.google.com/compute/docs/instances/preemptible?hl=ja)
    - 通常より遥かに低価格である代わりに以下の制約をうけるインスタンス
        - システムイベントにより、いつでも停止する可能性がある
        - 24 時間実行した後、必ず停止
        - 有限の Compute Engine リソースなので、常に利用できるわけではない
        - ライブ マイグレーションできない
        - サービスレベル契約（ SLA ）の非対象（ [Compute Engine SLA](https://cloud.google.com/compute/sla?hl=ja) ）
        - Google Cloud の無料枠に適用されない
    - バッチやフォールトトレラント（リトライ処理など強制終了に対応できる）なワークロードに適用する
- [Shielded VM](https://cloud.google.com/security/shielded-cloud/shielded-vm?hl=ja)
    - 以下の 3 つの機能をすべて有効にしたインスタンス
        - [仮想トラステッド プラットフォーム モジュール（vTPM）](https://cloud.google.com/security/shielded-cloud/shielded-vm?hl=ja#vtpm)
        - [整合性モニタリング](https://cloud.google.com/security/shielded-cloud/shielded-vm?hl=ja#integrity-monitoring)
        - [セキュアブート](https://cloud.google.com/security/shielded-cloud/shielded-vm?hl=ja#secure-boot)
    - インスタンスを起動した後、[Shielded VM のオプションを変更](https://cloud.google.com/compute/docs/instances/modifying-shielded-vm?hl=ja) するには、インスタンスを停止する必要がある

#### 1.1.1. インスタンステンプレート

[インスタンステンプレート](https://cloud.google.com/compute/docs/instance-templates?hl=ja) は、VM インスタンスと MIG を作成する際にテンプレートとして利用できるリソース。  
インスタンステンプレートで、マシンタイプ、イメージ、ラベルなどを定義できる。

```bash
$ gcloud compute instance-templates create example-template-custom \
    --machine-type e2-standard-4 \
    --image-family debian-9 \
    --image-project debian-cloud \
    --boot-disk-size 250GB
```

[その他のユースケース](https://cloud.google.com/compute/docs/instance-templates/create-instance-templates?hl=ja)

#### 1.1.2. マネージドインスタンスグループ（ MIG ）

インスタンステンプレートからインスタンスを起動・グループ化することにより、負荷分散やオートスケール、ヘルスチェックによる自動復旧を可能にする。  
なお、 MIG には **ゾーン MIG** と **リージョン（マルチゾーン） MIG** の 2 種類あり、可用性の観点から後者にすることが推奨される。

```bash
$ gcloud compute instance-groups managed create [インスタンスグループ名] \
    --base-instance-name=[作成されるインスタンス名] \
    --size=[インスタンス数] \
    --template=[インスタンステンプレート名] \
    --zome=[ゾーン名] \ # カンマ区切りで複数指定した場合、マルチゾーン MIG になる
    --region=[リージョン名] \ # リージョン（マルチゾーン） MIG になる
```

- 自動スケール（スケールイン・アウト）
    - [クールダウン期間](https://cloud.google.com/compute/docs/autoscaler?hl=ja#cool_down_period) （初期化期間）
        - 新しく作成されたインスタンスを自動スケーリングの判断に含めない時間
        - インスタンスが起動しても、サービスが起動していなければヘルスチェック NG になるため、このご検知を発生させないために設定（アプリの初期化時間をテストすることを推奨）
        - スケールアップの判定を遅らせ、必要以上に多くのインスタンスが作成されないようにするために設定する
    - [安定化期間](https://cloud.google.com/compute/docs/autoscaler?hl=ja#stabilization_period)
        - スケールインの判定のため、オートスケーラは直近10分間におけるピーク負荷に基づいてターゲットサイズを計算する（この 10 分を安定化期間という）
        - ピーク負荷後のスケールダウンに10分の余裕を持たせることで、頻繁なスケールアップ/ダウンの繰り返しを回避する
    - [オートスケーラーによる判断の理解](https://cloud.google.com/compute/docs/autoscaler/understanding-autoscaler-decisions?hl=ja)
- ヘルスチェック（自動修復のための）
    - 初期遅延
        - 自動修復のためのヘルスチェックが新しく作成したインスタンスを対象にしない時間
        - 初期化完了前にUnhealthyと判断され、無駄な再作成を防ぐ目的で設定する
    - [ヘルスチェックと自動修復の設定](https://cloud.google.com/compute/docs/instance-groups/autohealing-instances-in-migs?hl=ja)

### 1.2. マシンタイプ

- ざっくり CPU と メモリのセット
- 汎用（E2/N2/N2D/N1）、メモリ最適化（M1/M2）、コンピューティング最適化（C2）、アクセラレータ最適化（A2）など
- 割と自由に設定できる カスタムタイプ もある

提供されているマシンタイプは以下のコマンドで一覧取得できる。

```bash
$ gcloud compute machine-types list
WARNING: The following filter keys were not present in any resource : deprecated.state
NAME              ZONE                       CPUS  MEMORY_GB  DEPRECATED
a2-highgpu-1g     us-central1-a              12    85.00
a2-highgpu-2g     us-central1-a              24    170.00
a2-highgpu-4g     us-central1-a              48    340.00
（省略）
```

### 1.3. ストレージオプション

VM インスタンスからマウントして利用できるストレージには以下の種類がある。

- **ゾーン永続ディスク** : 効率的で信頼性の高い ブロックストレージ。
- **リージョン永続ディスク** : 2 つのゾーンに複製されたリージョン ブロックストレージ。
- **ローカル SSD** : 高パフォーマンスかつ一時的なローカル ブロックストレージ。
- **Cloud Storage バケット** : 手頃な料金のオブジェクトストレージ。
- **Filestore** : Google Cloud ユーザー向けの高性能ファイルストレージ。

永続ディスク、ローカル SSD はブロックストレージで ディスク としての扱いになる。  
さらに永続ディスクには以下の種類がある。

- **標準永続ディスク**（pd-standard）：標準のハードディスク ドライブ（HDD）
- **バランス永続ディスク**（pd-balanced）：ソリッド ステート ドライブ（SSD）、SSD 永続ディスクに比べ低コスト・低パフォーマンス
- **SSD 永続ディスク**（pd-ssd）：ソリッド ステート ドライブ（SSD）

永続ディスクは **ブートディスク** （OS イメージをインストールした起動ディスク）として利用することができ、 **スナップショット** もまた永続ディスクのみ対応する。（その他は無理）  
大まかな違いは以下。

|ストレージ|ユースケース|パフォーマンス|冗長性|スナップショット|バージョニング|ブートディスク|
|:---|:---|:---|:---|:---|:---|:---|
|永続ディスク|汎用|中|あり|あり|なし|○|
|ローカルSSD|高IOPS、低レイテンシ|高|なし|なし|なし|×|
|Cloud Storage バケット|低コスト、ゾーン間共有|低|あり|なし|あり|×|

### 1.4. イメージ

- イメージファミリーとそれに属する OS イメージからなる
- インスタンステンプレートや マネージドインスタンスグループ（ MIG ）から起動できる

イメージには以下の 2 種類ある。

- **公開イメージ**
    - Google、オープンソース コミュニティ、サードパーティ ベンダーによって提供され、保守されるブートディスクイメージ
- **カスタムイメージ**
    - ユーザーが所有し、アクセスを制御するブートディスクイメージ
    - [カスタム イメージの作成、削除、廃止](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images?hl=ja)

公開 OS イメージの情報は、以下のコマンドで取得可能。

```zsh
% gcloud compute images list
NAME                              PROJECT       FAMILY         DEPRECATED  STATUS
centos-6-v20181011                centos-cloud  centos-6                   READY
centos-7-v20181011                centos-cloud  centos-7                   READY
coreos-alpha-1953-0-0-v20181106   coreos-cloud  coreos-alpha               READY
coreos-beta-1939-1-0-v20181106    coreos-cloud  coreos-beta                READY
coreos-stable-1911-3-0-v20181106  coreos-cloud  coreos-stable              READY
(略)
```

### 1.5. スナップショット

- イメージより低コストで保存でき、バックアップに適する
- ディスクデータを複製できる点でイメージと似ているが、 **インスタンステンプレートからは起動できない**

### 1.6. ゲスト環境

- インスタンスを起動すると、ゲスト環境が自動的に VM インスタンスにインストールされる
- Compute Engine で VM を適切に実行するために、 **メタデータサーバー** のコンテンツを読み取る一連のスクリプト、デーモン、バイナリから構成される

### 1.7. メタデータサーバー

- クライアントからゲストオペレーティングシステムに情報を転送する通信チャネル
- すべてのインスタンスはメタデータをメタデータサーバーに保存（ `key-value` 形式）
- メタデータサーバーに対して、インスタンス内からも Compute Engine API からも、プログラムでクエリを実行できる
- [デフォルトの VM メタデータ値](https://cloud.google.com/compute/docs/metadata/default-metadata-values)

### 1.8. ライブマイグレーション

- Compute Engine では、ホストシステムでソフトウェアやハードウェアの更新などのイベントが発生しても、インスタンスの実行を継続できるように、ライブマイグレーションを提供している
- ユーザーが VM を再起動しなくても、実行中のインスタンスを同じゾーンの別のホストに移行できる
- ただし、以下および以下を含むインスタンスはライブマイグレーションできない
    - [Confidential VMs](https://cloud.google.com/compute/confidential-vm/docs/creating-cvm-instance)、GPU、[Cloud TPU](https://cloud.google.com/tpu/docs/tpus)、[プリエンプティブル インスタンス](https://cloud.google.com/compute/docs/instances/preemptible)

## 2. インスタンス作成

1. [gcloud コマンドの設定](#21-gcloud-コマンドの設定)
2. [インスタンス作成](#22-インスタンス作成)
3. [起動スクリプト](#23-起動スクリプト)
4. [単一テナントノードにインスタンス作成](#24-単一テナントノードにインスタンス作成)

### 2.1. gcloud コマンドの設定

gcloud コマンドの設定情報の確認と変更について記載する。

```bash
# 設定情報の確認（値が設定されている情報のみ）
$ gcloud config list

# 設定情報の確認（ unset も含めてすべて）
$ gcloud config list --all

# 特定の設定情報の確認
$ gcloud config list compute/region
$ gcloud config list core/project
$ gcloud config list project
$ gcloud config get-value project
```

なお、特定の設定情報を取得する場合、 `SECTION/PROPERTY` （たとえば `compute/region` ）の形式で指定するのだが、`SECTION` が `core` の場合は `core/` を省略できる。  
その他、 `SECTION/PROPERTY` に関する情報は [`gcloud config set`](https://cloud.google.com/sdk/gcloud/reference/config/set?hl=ja) を参照。  
情報の設定は `gcloud config set` コマンドを利用する。  
以下は、 gcloud コマンドで多用する情報のため、ターミナルセッション開始時に設定することを推奨する。

```bash
# `--project` オプションを省略可能にする
$ gcloud config set project <PROJECT ID>

# `--region` オプションを省略可能にする
$ gcloud config set compute/region <REGION>

# `--zone` オプションを省略可能にする
$ gcloud config set compute/zone <ZONE>
```

`gcloud` コマンドで設定したデフォルト値は `bq` や `gsutil` などの **Cloud SDK で共有される** 。（全てかどうかはわからない、、、）  
なお、PROJECT 、 REGION 、 ZONE の選択肢となる一覧情報は以下のコマンドで取得可能。

```bash
# プロジェクト一覧
$ gcloud projects list

# リージョン一覧
$ gcloud compute regions list

# ゾーン一覧
$ gcloud compute zones list
```

また、 `gcloud compute` ではメタデータを持っており、 `project-info` で参照できる。

```bash
$ gcloud compute project-info describe --project <PROJECT ID>
# 例えば、以下のようなメタデータを持っている（持っていないときもある）
# リージョン情報：google-compute-default-region
# ゾーン情報　　：google-compute-default-zone
```

また、コマンドでの設定ではなく環境変数を利用してもいい。

```bash
$ export PROJECT_ID=<your_project_ID>
$ export ZONE=<your_zone>
```

### 2.2. インスタンス作成

[gcloud compute instances create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create?hl=ja)

```bash
$ gcloud compute instances create INSTANCE_NAME --machine-type n1-standard-2

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-7b6d0cd2d87a/zones/us-central1-c/instances/gcelab2].
NAME           ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
INSTANCE_NAME  us-central1-c  n1-standard-2               10.128.0.3   34.70.76.162  RUNNING
```

- イメージ（ `--image=IMAGE` | `--image-family=IMAGE_FAMILY` ）
    - イメージもしくはイメージファミリを指定する必要があり、両方省略した場合はイメージファミリのデフォルト値「Debian 10 (buster) 」が適用される
    - イメージファミリのみ指定された場合は、ファミリ内の非推奨でない最新のイメージが的ようされる
- ブートディスク
    - インスタンスと同じ名前のブートディスク（ 10 GB のバランス永続ディスク）が自動的に作成され組み込まれる
    - なお、デフォルトではインスタンスを削除するとブートディスクも削除される

上記コマンドで作成したら、下記コマンドで確認する。

```zsh
% gcloud compute instances describe INSTANCE_NAME
```

なお、プロジェクト毎に `default` という名前のデフォルト VPC ネットワークが作成されており、ネットワークの詳細を指定せずに VM を作成すると、デフォルト VPC ネットワークと同じリージョンにあるサブネットにインスタンスが作成される。  
以下のオプションを指定することで、対象のネットワークにインスタンスが作成される。

- `--subnet=SUBNET_NAME`: サブネットの名前。 VPC は指定したサブネットから推測される。
- `--zone=ZONE_NAME`: VM を作成するゾーン（ `europe-west1-b` など）。リージョンは指定したゾーンから推測される。

また、以下のようなオプションもある。

- **Shielded VM インスタンス** で作成： `--shielded-secure-boot` オプション
- **プリエンプティブルインスタンス** で作成： `--preemptible` オプション

### 2.3. 起動スクリプト

メタデータ（ `--metadata` オプション）に起動スクリプトの情報を付与して起動時に実行させることができる。

```bash
$ gcloud compute instances create www1 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"
```

起動スクリプトが長いなどと行った場合、 Cloud Storage バケットに配置してあるスクリプトを実行することも可能。

```bash
# Cloud Storage バケットの作成
$ gsutil mb -c regional -l us-central1 gs://challenge-quest-12345
# Cloud Console でスクリプトをアップロード

# インスタンスの作成
$ gcloud compute instances create apache --scopes storage-ro \
  --metadata startup-script-url=gs://challenge-quest-12345/resources-install-web.sh
```

`--scopes storage-ro` オプションを付与することで、インスタンスのサービスアカウントに Cloud Storage へのアクセス権を付与できる。

### 2.4. 単一テナントノードにインスタンス作成

https://cloud.google.com/compute/docs/nodes/provisioning-sole-tenant-vms?hl=ja

#### 2.4.1. 単一テナントノード テンプレートを作成

[単一テナントノード](https://cloud.google.com/compute/docs/nodes/sole-tenant-nodes?hl=ja) のテンプレートを作成する。

```zsh
% gcloud compute sole-tenancy node-templates create TEMPLATE_NAME \
  --node-type=NODE_TYPE \
  [--node-affinity-labels=AFFINITY_LABELS \]
  [--region=REGION \]
  [--accelerator type=GPU_TYPE,count=GPU_COUNT \]
  [--disk type=local-ssd,count=DISK_COUNT,size=DISK_SIZE]
```

#### 2.4.2. 単一ノードグループの自動スケーリング

```zsh
% gcloud compute sole-tenancy node-groups create group-name \
  --node-template template-name \
  --target-size size \
  --maintenance-policy maintenance-policy \
  --zone zone \
  --autoscaler-mode mode \
  --max-nodes max-nodes \
  --min-nodes min-nodes
```

## 3. ディスクの作成とアタッチ

- [ディスクの作成とアタッチ](https://cloud.google.com/compute/docs/disks/add-persistent-disk)

ディスクの作成は以下。

```bash
$ gcloud compute disks create DISK_NAME --size DISK_SIZE --type DISK_TYPE
```

作成したディスクは以下でインスタンスにアタッチする。

```bash
$ gcloud compute instances attach-disk INSTANCE_NAME --disk DISK_NAME
```
  
なお、アタッチしたディスクはマウントしないと利用できないことに注意。  
また、インスタンス作成時（ `gcloud compute instances create` ）にディスクをアタッチした状態で起動するには以下の方法がある。

- 作成済みのディスクをアタッチして起動（ [参考](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create#--disk) ）
    - オプション `--disk=[auto-delete=AUTO-DELETE],[boot=BOOT],[device-name=DEVICE-NAME],[mode=MODE],[name=NAME],[scope=SCOPE]`
- ディスクを作成・アタッチして起動（ [参考](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create#--create-disk) ）
    - オプション `--create-disk=[PROPERTY=VALUE,…]`

## 4. ファイアウォールルールの作成

以下のコマンドでファイアウォールルールを作成する。

```bash
$ gcloud compute firewall-rules create allow-http \
    --action=allow \
    --direction=ingress \
    --source-ranges=0.0.0.0/0 \
    --target-tags=http-server \
    --rules=tcp:80
```

なお、ファイアウォールルールはインスタンスに付与された **ネットワークタグ** や **サービスアカウント** によって適用され、上記の `--target-tags` や `--target-service-accounts` オプションで設定された値と対応する。  
インスタンス作成時（ `gcloud compute instances create` ）に `--tags` オプションを用いてネットワークタグを付与できるが、以下のコマンドで既存インスタンスにタグ付けすることもできる。

```bash
$ gcloud compute instances add-tags INSTANCE_NAME --tags=http-server
```

## 5. インスタンスへの接続

Cloud Console にて「サイドメニュー」->「Compute Engine」->「 VM インスタンス」と選択し、該当インスタンスのレコードで「 SSH 」をクリックするとポップアップしてターミナルが立ち上がる。  
もしくは、以下のコマンドを実行する。

```bash
# USEGE
$ gcloud compute ssh [インスタンス名] --project=PROJECT_ID --zone=ZONE VM_NAME

# 具体例（Pass Phrase は空でいい）
$ gcloud compute ssh gcelab2 --zone us-central1-c
```

接続できたら、以下のコマンドの管理者権限になる。

```bash
$ sudo su -
```

ちなみに Linux は SSH 、 Windows は RDP で接続する。

## 6. Cloud Monitoring / Cloud Logging 連携

Cloud Monitoring では、クラウドで実行されるアプリケーションのパフォーマンスや稼働時間、全体的な動作状況を確認できる。  
Google Cloud、Amazon Web Services、ホストされた稼働時間プローブ、アプリケーション インストゥルメンテーション、よく使われるさまざまなアプリケーション コンポーネント（Cassandra、Nginx、Apache ウェブサーバー、Elasticsearch など）から、指標、イベント、メタデータを収集する。  
これらのデータを取り込んでダッシュボード、グラフ、アラートを介して分析情報を提供する。  
Cloud Monitoring のアラート機能を Slack、PagerDuty、HipChat、Campfire などに組み込むこともできる。

インスタンスから情報を収集する VM に [Monitoring エージェント](https://cloud.google.com/monitoring/agent) と [Logging エージェント](https://cloud.google.com/logging/docs/agent?hl=ja) をインストールすることで収集可能となる。  
対象の VM インスタンスに SSH した後、以下のコマンドでインストールする。

```bash
# Monitoring エージェントのインストール
$ curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
$ sudo bash add-monitoring-agent-repo.sh
$ sudo apt-get update
$ sudo apt-get install stackdriver-agent

# Logging エージェント
$ curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh
$ sudo bash add-logging-agent-repo.sh
$ sudo apt-get update
$ sudo apt-get install google-fluentd
```

また、Monitoring ワークスペースを作成する必要がある。

- Cloud Console で、ナビゲーション メニュー > [モニタリング] をクリックすると、ワークスペースがプロビジョニングされる
- 左側のメニューで [稼働時間チェック] をクリックし、[稼働時間チェックの作成] をクリック
- 左側のメニューで [アラート]、[Create Policy] の順にクリック
- ナビゲーション メニュー > [Logging] > [ログビューア]

## 7. サービスアカウント

VM インスタンスにサービスアカウントを付与することによってリソースへのアクセス制御を行う。  
サービスアカウントは以下のように作成する。

```bash
# サービスアカウントの作成
$ gcloud iam service-accounts create SERVICE_ACCOUNT_ID --display-name="DISPLAY_NAME"

# サービスアカウントへのロールの付与
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:SERVICE_ACCOUNT_ID@PROJECT_ID.iam.gserviceaccount.com" \
    --role="ROLE_NAME"
```

VM インスタンス作成時にサービスアカウントを付与するには以下。

```bash
$ gcloud compute instances create INSTANCE_NAME \
    --service-account=SERVICE_ACCOUNT_EMAIL \
    [--scopes=[SCOPES,...]]
```

既存の VM インスタンスにサービスアカウントを付与するのは以下。

```bash
$ gcloud compute instances set-service-account INSTANCE_NAME \
    --service-account=SERVICE_ACCOUNT \
    [--scopes=[SCOPE,...]
```

- [ベストプラクティス](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances?hl=ja#best_practices)
    - Compute Engine のデフォルトのサービス アカウントを使用せずに、新しいサービス アカウントを作成
    - 必要なリソースについてのみ、このサービス アカウントに IAM ロールを付与
    - このサービス アカウントとして実行するようにインスタンスを構成
    - インスタンスの https://www.googleapis.com/auth/cloud-platform スコープにすべての Google Cloud APIs への完全アクセス権を与えるには、インスタンスの IAM 権限をサービス アカウントの IAM のロールで完全に決定するようにする

## 8. コマンド整理

### 8.1. Apache の VM インスタンスを作成

- Linux 仮想マシン インスタンスの作成
- VM インスタンスへの公開アクセスの有効化
- 基本的な Apache ウェブサーバーの実行
- サーバーのテスト

```bash
# ゾーンの設定
$ gcloud config set compute/zone us-central1-a

# インスタンスの作成
$ gcloud compute instances create apache --machine-type n1-standard-2

# インスタンスへタグ追加
$ gcloud compute instances add-tags apache --tags=http-server

# ファイアウォールルールの作成
$ gcloud compute firewall-rules create allow-http \
    --action=allow \
    --direction=ingress \
    --source-ranges=0.0.0.0/0 \
    --target-tags=http-server \
    --rules=tcp:80

# SSH して apache インストール
$ gcloud compute ssh apache
$ sudo su -
$ apt-get update
$ apt-get install apache2 -y
$ service apache2 restart
```

### 8.2. Apache の VM インスタンスを作成（起動スクリプト利用）

```bash
# ゾーンの設定
$ gcloud config set compute/zone us-central1-a

# Cloud Storage バケットの作成
$ gsutil mb -c regional -l us-central1 gs://challenge-quest-12345
# Cloud Console でスクリプトをアップロード

# インスタンスの作成
$ gcloud compute instances create apache --scopes storage-ro \
  --metadata startup-script-url=gs://challenge-quest-12345/resources-install-web.sh

# ファイアウォールルールの作成
$ gcloud compute firewall-rules create allow-http \
    --action=allow \
    --direction=ingress \
    --source-ranges=0.0.0.0/0 \
    --target-tags=http-server \
    --rules=tcp:80

# インスタンスへタグ追加
$ gcloud compute instances add-tags apache --tags=http-server
```

ちなみに `resources-install-web.sh` は以下。

```bash
#!/bin/bash
apt-get update
apt-get install -y apache2
```