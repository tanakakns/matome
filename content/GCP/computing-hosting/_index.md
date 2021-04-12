---
title: "コンピューティングとホスティング"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

1. [サービス一覧](#1-サービス一覧)
2. [Compute Engine](#2-compute-engine)
3. [Google Kubernetes Engine](#3-google-kubernetes-engine)

<!--more-->

## 1. サービス一覧

[Google Cloud のコンピューティング プロダクト](https://cloud.google.com/products/compute)

|サービス|モデル|形式|ユースケース|対応AWS|
|:---|:---|:---|:---|:---|
|Compute Engine / GCE|IaaS|VMイメージ|一般的なワークロード|EC2|
|Kubernetes Engine / GKE|IaaS/PaaS|クラスタ|コンテナワークロード|EKS/ECS|
|Cloud Run|PaaS|コンテナ|コンテナワークロード|Fargate|
|App Engine フレキシブル環境|PaaS|アプリ/コンテナ|HTTPサービス/バックエンドアプリ|Elastic Beanstalk|
|App Engine スタンダード環境|PaaS|アプリ|HTTPサービス/バックエンドアプリ|Elastic Beanstalk|
|Cloud Functions|FaaS|関数|イベントドリブンな操作|Lambda|

## 2. Compute Engine

[公式ガイド](https://cloud.google.com/compute/docs/how-to?hl=ja)

**Compute Engine** は Google のインフラストラクチャ上に仮想マシンを構築するサービス。

1. [コンセプト](#2-1-コンセプト)
2. [インスタンス作成](#2-2-インスタンス作成)
3. [単一テナントノードにインスタンス作成](#2-3-単一テナントノードにインスタンス作成)
4. [インスタンスへの接続](#2-4-インスタンスへの接続)
5. Cloud Monitoring

※ [Compute Engine のリージョン選択に関するベスト プラクティス](https://cloud.google.com/solutions/best-practices-compute-engine-region-selection?hl=ja)

### 2.1. コンセプト

- VM インスタンス
    - マシンタイプとストレージからなる
    - VM オプションとして以下のようなものがある
        - 可用性ポリシー
        - サービスアカウント
        - プリエンプティブル VM
        - Shield VM
- マシンタイプ
    - ざっくり CPU と メモリのセット
    - 汎用（E2/N2/N2D/N1）、メモリ最適化（M1/M2）、コンピューティング最適化（C2）、アクセラレータ最適化（A2）など
    - 割と自由に設定できる カスタムタイプ もある
- ストレージオプション
    - 永続ディスク、ローカルディスク、Cloud Storageバケット などから選択できる
    - また、複製のために、イメージ、スナップショットが利用できる
- イメージ
    - イメージファミリーとそれに属する OS イメージからなる
    - インスタンステンプレートや マネージドインスタンスグループ（ MIG ）から起動できる
- スナップショット
    - イメージより低コストで保存でき、バックアップに適する
    - ディスクデータを複製できる点でイメージと似ているが、インスタンステンプレートからは起動できない
- ゲスト環境
    - インスタンスを起動すると、ゲスト環境が自動的に VM インスタンスにインストールされる
    - Compute Engine で VM を適切に実行するために、メタデータサーバーのコンテンツを読み取る一連のスクリプト、デーモン、バイナリから構成される
- メタデータサーバー
    - クライアントからゲストオペレーティングシステムに情報を転送する通信チャネル
    - すべてのインスタンスはメタデータをメタデータサーバーに保存（ `key-value` 形式）
    - メタデータサーバーに対して、インスタンス内からも Compute Engine API からも、プログラムでクエリを実行できる
    - [デフォルトの VM メタデータ値](https://cloud.google.com/compute/docs/metadata/default-metadata-values)
- ライブマイグレーション
    - Compute Engine では、ホストシステムでソフトウェアやハードウェアの更新などのイベントが発生しても、インスタンスの実行を継続できるように、ライブマイグレーションを提供している
    - ユーザーが VM を再起動しなくても、実行中のインスタンスを同じゾーンの別のホストに移行できる
    - ただし、以下および以下を含むインスタンスはライブマイグレーションできない
        - [Confidential VMs](https://cloud.google.com/compute/confidential-vm/docs/creating-cvm-instance)、GPU、[Cloud TPU](https://cloud.google.com/tpu/docs/tpus)、[プリエンプティブル インスタンス](https://cloud.google.com/compute/docs/instances/preemptible)

#### 2.1.1. VM オプション

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


#### 2.1.2. ストレージオプション

|ストレージ|ユースケース|パフォーマンス|冗長性|スナップショット|バージョニング|ブートディスク|
|:---|:---|:---|:---|:---|:---|:---|
|永続ディスク|汎用|中|あり|あり|なし|○|
|ローカルSSD|高IOPS、低レイテンシ|高|なし|なし|なし|×|
|Cloud Storage バケット|低コスト、ゾーン間共有|低|あり|なし|あり|×|

#### 2.1.3. マネージドインスタンスグループ（ MIG ）

インスタンステンプレートからインスタンスを起動・グループ化することにより、負荷分散やオートスケール、ヘルスチェックによる自動復旧を可能にする。  
可用性の観点から **マルチゾーン MIG** にすることが推奨される。

```bash
$ gcloud compute instance-groups managed create [インスタンスグループ名] \
    --base-instance-name=[作成されるインスタンス名] \
    --size=[インスタンス数] \
    --template=[インスタンステンプレート名] \
```

#### 2.1.4. インスタンステンプレート

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

### 2.2. インスタンス作成

#### 2.2.1. デフォルト値の設定

まずは、デフォルト値を以下のコマンドで確認する。

```bash
# USAGE
$ gcloud compute project-info describe --project <your_project_ID>

# 具体例
$ gcloud compute project-info describe --project qwiklabs-gcp-00-97e14e7c9a36
commonInstanceMetadata:
  fingerprint: e-kQ-2nHXX0=
  items:
  - key: ssh-keys
    value: student-00-76522ecf0729:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC499dU9Bd3DvUOav+TicjmG5p0S6ip2z4gCpjiKjiqxTeeAT233lGxBbZ0MHrBGoZRWkD9OSMJ2Bpt//DFUmXDKI57ueSF7w0DiiKZ
/aMd3uVC2BNk8yUcCgK5PMXCs/tOvUHDgffXPoxYYJFiUeVqvBhfSKiF9GnARJegPDFB+ox7QGhW2vfPvUiOEl5fyu/qKBu5IG0lNnLC/24J6+ogF/Ri3nUjt8PtuBUKRjlJyky0LQ+c5gnqM2g2cRqgCB15T5DO8w3edOPyrre3qMT
ZFNQjEqpx9NU3ABlLM/0LVPeksmXr/FgyT7nZDrkrvryd9YnkaN58gnndhwAsBz49
      student-00-76522ecf0729@qwiklabs.net
  - key: enable-oslogin
    value: 'true'
  - key: google-compute-default-zone   # デフォルトのゾーン
    value: us-central1-a
  - key: google-compute-default-region # デフォルトのリージョン
    value: us-central1
  kind: compute#metadata
creationTimestamp: '2021-04-05T00:24:25.019-07:00'
defaultNetworkTier: PREMIUM
defaultServiceAccount: 123007011495-compute@developer.gserviceaccount.com
id: '8530956731357267399'
kind: compute#project
name: qwiklabs-gcp-00-97e14e7c9a36
quotas:
- limit: 1000.0
  metric: SNAPSHOTS
（省略）
- limit: 15.0
  metric: INTERNAL_TRAFFIC_DIRECTOR_FORWARDING_RULES
  usage: 0.0
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-97e14e7c9a36
xpnProjectStatus: UNSPECIFIED_XPN_PROJECT_STATUS
```

以下のコマンドのデフォルトリージョンの設定。

```bash
$ gcloud config set compute/region <REGION>
```

以下のコマンドのデフォルトゾーンの設定。（なお、ゾーンにリージョン情報も含まれるため、ゾーンリソースの場合はこれだけでもいいかも）

```bash
$ gcloud config set compute/zone <ZONE>
```

また、コマンドでの設定ではなく以下の環境変数でも設定可能。

```bash
$ export PROJECT_ID=<your_project_ID>
$ export ZONE=<your_zone>
```

また、現在使用している環境の構成は以下で取得。

```bash
$ gcloud config list

[component_manager]
disable_update_check = True
[compute]
gce_metadata_read_timeout_sec = 30
[core]
account = student-00-76522ecf0729@qwiklabs.net
disable_usage_reporting = True
project = qwiklabs-gcp-00-97e14e7c9a36
[metrics]
environment = devshell
Your active configuration is: [cloudshell-5813]

# すべての構成を取得する場合
$ gcloud config list --all

[accessibility]
screen_reader (unset)
[ai]
region (unset)
[ai_platform]
region (unset)
[app]
cloud_build_timeout (unset)
promote_by_default (unset)
stop_previous_version (unset)
（省略）
```

#### 2.2.2. ブート永続ディスク

ブート永続ディスクは、OS イメージが格納されている小規模なディスクで大きく以下の 2 種類ある。

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

#### 2.2.3. インスタンス作成

[gcloud compute instances create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create?hl=ja)

```bash
# USAGE
$ gcloud compute instances create INSTANCE_NAMES [INSTANCE_NAMES …] \
    [--image=IMAGE | --image-family=IMAGE_FAMILY] \
    --image-project=IMAGE_PROJECT

# 具体例
$ gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-c

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-7b6d0cd2d87a/zones/us-central1-c/instances/gcelab2].
NAME     ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
gcelab2  us-central1-c  n1-standard-2               10.128.0.3   34.70.76.162  RUNNING
# default で以下が適用されている
# - Debian 10 (buster)
# - インスタンスと同じ名前のルート永続ディスクが自動的にインスタンスに組み込まれる
```

- `VM_NAME` ：インスタンス名
- `IMAGE` （公開イメージの必須バージョン） or `IMAGE_FAMILY` （イメージファミリ名）
    - `IMAGE_FAMILY` を指定した場合、ファミリ内の非推奨でない最新の `IMAGE` が適用される
- `IMAGE_PROJECT` ：イメージプロジェクト名

上記コマンドで作成したら、下記コマンドで確認する。

```zsh
% gcloud compute instances describe INSTANCE_NAMES
```

なお、プロジェクト毎に `default` という名前のデフォルト VPC ネットワークが作成されており、ネットワークの詳細を指定せずに VM を作成すると、デフォルト VPC ネットワークと同じリージョンにあるサブネットにインスタンスが作成される。  
以下のオプションを指定することで、対象のネットワークにインスタンスが作成される。

- `--subnet=SUBNET_NAME`: サブネットの名前。ネットワークは指定したサブネットから推測される。
- `--zone=ZONE_NAME`: VM を作成するゾーン（ `europe-west1-b` など）。リージョンはこのゾーンから推測される。

コマンドの具体例は以下。

```bash
$ gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-c

Created [...gcelab2].
NAME     ZONE           MACHINE_TYPE  ...    STATUS
gcelab2  us-central1-c  n1-standard-2 ...    RUNNING
```

上記のコマンドで作成されたインスタンスには以下のデフォルト値が適用されている。

- 最新の Debian 10（buster）イメージ
- マシンタイプは n1-standard-2
- インスタンスと同じ名前のルート永続ディスクが自動的にインスタンスに組み込まれる

その後、 以下のコマンドで、SSH を使用してインスタンスに接続することもできる。

```bash
$ gcloud compute ssh gcelab2 --zone us-central1-c
```

#### 2.2.4. Shielded VM インスタンス作成

オプションで `--shielded-secure-boot` フラグを指定すると、Shielded VM 機能をすべて有効にしたインスタンスを作成する。

#### 2.2.5. プリエンプティブルインスタンス作成

```zsh
% gcloud compute instances create [INSTANCE_NAME] --preemptible
```

### 2.3. 単一テナントノードにインスタンス作成

https://cloud.google.com/compute/docs/nodes/provisioning-sole-tenant-vms?hl=ja

#### 2.3.1. 単一テナントノード テンプレートを作成

[単一テナントノード](https://cloud.google.com/compute/docs/nodes/sole-tenant-nodes?hl=ja) のテンプレートを作成する。

```zsh
% gcloud compute sole-tenancy node-templates create TEMPLATE_NAME \
  --node-type=NODE_TYPE \
  [--node-affinity-labels=AFFINITY_LABELS \]
  [--region=REGION \]
  [--accelerator type=GPU_TYPE,count=GPU_COUNT \]
  [--disk type=local-ssd,count=DISK_COUNT,size=DISK_SIZE]
```

#### 2.3.2. ノードグループの自動スケーリング

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

### 2.4. インスタンスへの接続

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

### 2.5. Cloud Monitoring

Cloud Monitoring では、クラウドで実行されるアプリケーションのパフォーマンスや稼働時間、全体的な動作状況を確認できる。  
Google Cloud、Amazon Web Services、ホストされた稼働時間プローブ、アプリケーション インストゥルメンテーション、よく使われるさまざまなアプリケーション コンポーネント（Cassandra、Nginx、Apache ウェブサーバー、Elasticsearch など）から、指標、イベント、メタデータを収集する。  
これらのデータを取り込んでダッシュボード、グラフ、アラートを介して分析情報を提供する。  
Cloud Monitoring のアラート機能を Slack、PagerDuty、HipChat、Campfire などに組み込むこともできる。

インスタンスから情報を収集する VM に [Monitoring エージェント](https://cloud.google.com/monitoring/agent) と Logging エージェントをインストールすることで収集可能となる。  
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

Cloud Console で、ナビゲーション メニュー > [モニタリング] をクリックすると、ワークスペースがプロビジョニングされる。  
左側のメニューで [稼働時間チェック] をクリックし、[稼働時間チェックの作成] をクリック。  
左側のメニューで [アラート]、[Create Policy] の順にクリック。  
ナビゲーション メニュー > [Logging] > [ログビューア] 。

### 2.6. チャレンジクエスト

#### 2.6.1. Apache の VM インスタンスを作成

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

### 2.6.2. Apache の VM インスタンスを作成（起動スクリプト利用）

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

## 3. Google Kubernetes Engine

1. [コンセプト](#3-1-コンセプト)
2. [クラスタ作成](#3-2-クラスタ作成)
3. [デプロイ管理](#3-3-デプロイ管理)
4. Spinnaker と Kubernetes Engine を使用した継続的デリバリー パイプライン

### 3.1. コンセプト

### 3.2. クラスタ作成

デフォルトのコンピューティング ゾーンを設定する。

```bash
$ gcloud config set compute/zone us-central1-a

Updated property [compute/zone].
```

GKE クラスタを作成する。

```bash
# USAGE
$ gcloud container clusters create [CLUSTER-NAME]

# 具体例
$ gcloud container clusters create my-cluster
Creating cluster my-cluster in us-central1-a... Cluster is being health-checked (master is healthy)...done.
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-47d564cd39d1/zones/us-central1-a/clusters/my-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-a/my-cluster?project=qwiklabs-gcp-01-47d564cd39d1
kubeconfig entry generated for my-cluster.
NAME        LOCATION       MASTER_VERSION   MASTER_IP      MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
my-cluster  us-central1-a  1.18.16-gke.302  34.67.190.125  e2-medium     1.18.16-gke.302  3          RUNNING
```

クラスタの認証情報を取得する。（ kubeconfig に設定が追加される）

```bash
# USAGE
$ gcloud container clusters get-credentials [CLUSTER-NAME]

# 具体例
$ gcloud container clusters get-credentials my-cluster
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-cluster.
```

試しに作成したクラスタにコンテナをデプロイする。

```bash
$ kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
deployment.apps/hello-server created

$ kubectl expose deployment hello-server --type=LoadBalancer --port 8080
service/hello-server exposed

$ kubectl get service
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
hello-server   LoadBalancer   10.3.245.136   35.238.58.233   8080:30372/TCP   70s
kubernetes     ClusterIP      10.3.240.1     <none>          443/TCP          10m

$ curl http://35.238.58.233:8080
Hello, world!
Version: 1.0.0
Hostname: hello-server-57684579f-kw54q
```

クラスタを削除する。

```bash
# USAGE
$ gcloud container clusters delete [CLUSTER-NAME]

# 具体例
$ gcloud container clusters delete my-cluster
The following clusters will be deleted.
 - [my-cluster] in [us-central1-a]
Do you want to continue (Y/n)?  Y
Deleting cluster my-cluster...done.
Deleted [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-47d564cd39d1/zones/us-central1-a/clusters/my-cluster].
```

### 3.3. デプロイ管理

#### 3.3.1. Deployment オブジェクトの詳細

`kubectl explain` コマンドを使用すると、Deployment などのオブジェクトに関する情報が取得できる。

```bash
$ kubectl explain deployment

# 全てのフィールドを確認する場合
$ kubectl explain deployment --recursive

# 特定のフィールドを確認する場合
$ kubectl explain deployment.metadata.name
```

#### 3.3.2. Deployment の作成

Google 提供の資材を使って、Cloud Console の Cloud Shell 上で実施していく。  
まずはクラスタを作成する。

```bash
# デフォルトのゾーンを設定
$ gcloud config set compute/zone us-central1-a

# サンプルコードを入手
$ gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
$ cd orchestrate-with-kubernetes/kubernetes

# 5 つの n1-standard-1 ノードを含むクラスタを作成
$ gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"

NAME      LOCATION       MASTER_VERSION   MASTER_IP     MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
bootcamp  us-central1-a  1.18.16-gke.302  35.238.203.7  e2-medium     1.18.16-gke.302  5          RUNNING
```

`deployments/auth.yaml` の `image` を以下のように変更する。

```yaml
…
containers:
- name: auth
  image: kelseyhightower/auth:1.0.0
...
```

Deployment でレプリカが 1 つ作成され、バージョン 1.0.0 の auth コンテナが使用される。  
`kubectl create` を使用して Deployment オブジェクトを作成する。

```bash
$ kubectl create -f deployments/auth.yaml

# 確認
$ kubectl get deployments
$ kubectl get replicasets
$ kubectl get pods
```

次に、auth デプロイ用のサービスを作成する。

```bash
$ kubectl create -f services/auth.yaml
```

ここまでで `auth` を作成したが、 `hello` 、 `frontend` も作成していく。

```bash
# hello
$ kubectl create -f deployments/hello.yaml
$ kubectl create -f services/hello.yaml

# frontend
$ kubectl create secret generic tls-certs --from-file tls/
$ kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
$ kubectl create -f deployments/frontend.yaml
$ kubectl create -f services/frontend.yaml
```

`frontend` の外部 IP を確認して `curl` で動確する。

```bash
$ kubectl get services frontend
NAME       TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)         AGE
frontend   LoadBalancer   10.115.243.177   34.66.110.111   443:31848/TCP   46s

$ curl -ks https://34.66.110.111
{"message":"Hello"}
```

#### 3.3.3. スケールアウト・イン

Deployment をスケーリングするには、 `spec.replicas` フィールドを更新する。  
`kubectl explain` コマンドで、このフィールドの説明を確認してみる。

```bash
$ kubectl explain deployment.spec.replicas
KIND:     Deployment
VERSION:  apps/v1
FIELD:    replicas <integer>
DESCRIPTION:
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.
```

`kubectl scale` コマンドでスケールアウト・インする。

```bash
# Pod 数を 5 へスケールアウト
$ kubectl scale deployment hello --replicas=5

# 確認
$ kubectl get pods | grep hello- | wc -l
5

# Pod 数を 3 へスケールイン
$ kubectl scale deployment hello --replicas=3

# 確認
$ kubectl get pods | grep hello- | wc -l
3
```

#### 3.3.3. ローリング アップデート

ReplicaSet が新たに作成され、古い ReplicaSet のレプリカ数が減少するにつれて、新しい ReplicaSet のレプリカ数が徐々に増加する形で実行される。  
Deployment を更新するには、次のコマンドを実行してマニフェストを編集する。

```bash
$ kubectl edit deployment hello
# image を「kelseyhightower/hello:1.0.0」から「kelseyhightower/hello:2.0.0」へ変更
```

新しい ReplicaSet の確認、および、ロールアウト履歴にも新しいエントリがあるか確認する。

```bash
$ kubectl get replicaset
NAME                  DESIRED   CURRENT   READY   AGE
auth-7cffdb8677       1         1         1       14m
frontend-7b4b97c4dc   1         1         1       12m
hello-687bb4b8f4      0         0         0       13m
hello-d99c798f        3         3         3       60s

$ kubectl rollout history deployment/hello
deployment.apps/hello
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

ロールアウトの状況は以下のコマンドで確認できる。

```bash
$ kubectl rollout status deployment/hello

# もしくは

$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

ロールアウトの状況が芳しくない場合、以下のコマンドで停止・再開ができる。


```bash
# ロールアウト停止
$ kubectl rollout pause deployment/hello

# ロールアウト再開
$ kubectl rollout resume deployment/hello
```

なお、なんらか問題が発生して、ロールバックが必要な場合は以下のコマンドで実施できる。

```bash
# ロールバック
$ kubectl rollout undo deployment/hello

# ロールアウト履歴確認
$ kubectl rollout history deployment/hello
deployment.apps/hello
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

# マニフェストを確認（2.0.0 -> 1.0.0 にもどってる
$ kubectl edit deployment hello

# 全ての Pod がもどっているか確認
$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

#### 3.3.4. カナリア デプロイ

カナリア デプロイでは、一部のユーザーに変更をリリースすることで、新しいリリースに伴うリスクを軽減する手法。  
新しいバージョンの Deployment を作成する。

```bash
$ kubectl create -f deployments/hello-canary.yaml

# 確認
$ kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
auth           1/1     1            1           28m
frontend       1/1     1            1           26m
hello          3/3     3            3           27m
hello-canary   0/1     1            0           8s  # 新しい hello
```

この状態だと、単純に新しい別の Deployment を作成しただけに見えるが、 hello-canary の Label に `app:hello` が設定されており、これは hello service のセレクタになっているため、リクエストがルーティングされ、カナリアデプロイされた状態になっている。  
以下のコマンドを何度が実行していると、レスポンスが異なっており、カナリアデプロイされていることがわかる。

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
# {"version":"1.0.0"} or {"version":"2.0.0"}
```

ユーザーをいずれかのデプロイに「固定」したい場合、 **セッション アフィニティ** を指定してサービスを作成することで実現できる。  
以下の例では Client の IP でルーティングを固定している。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  sessionAffinity: ClientIP
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
```

#### 3.3.5. Blue / Green デプロイ

Kubernetes は、古い「Blue」バージョン用と新しい「Green」バージョン用に 2 つの異なるデプロイを作成することでこれを実現します。  
このデプロイには、ルータとして機能するサービスを介してアクセスする。  
新しい「Green」バージョンが稼働したら、サービスを更新してそのバージョンを使用するように切り替えることにより実現する。

既存の hello サービスを、`app:hello` 、 `version: 1.0.0` のセレクタを持つように更新する。（元々は `app:hello` だけ）

```bash
$ kubectl apply -f services/hello-blue.yaml
# resource service/hello is missing という警告は無視
```

Green の Deployment をデプロイする。（元々の hello は Blue の扱いで image のバージョンは 1.0.0）

```bash
$ kubectl create -f deployments/hello-green.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
```

現状は Blue （つまり 1.0.0）であることを確認する。

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
# 何度叩いても 1.0.0
```

次に新しいバージョン（つまり Green ）を指定するようにサービスを更新する。（ セレクタが `version: 2.0.0` になっている）

```bash
$ kubectl apply -f services/hello-green.yaml
```

`curl` で確認すると今度は `2.0.0` しか返ってこない。（つまり、 Blue -> Green になった！）

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

ロールバックしたいときは再度 service を Blue に戻せばいい。（つまり、セレクタを `version: 1.0.0` にするだけ）

```bash
$ kubectl apply -f services/hello-blue.yaml

# 確認
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
# 1.0.0 しか返ってこない
```

### 3.4. Spinnaker と Kubernetes Engine を使用した継続的デリバリー パイプライン

まずは環境を設定する。

```bash
# ゾーンの設定
$ gcloud config set compute/zone us-central1-f

# プロジェクトを確認して環境変数へ
$ gcloud config list project
[core]
project = qwiklabs-gcp-04-4ceec88343aa
$ export PROJECT=qwiklabs-gcp-04-4ceec88343aa

# クラスタ作成
$ gcloud container clusters create spinnaker-tutorial \
    --machine-type=n1-standard-2
```

Spinnaker 用のサービスアカウントを作成して Cloud Storage にデータを保存できるようにする。

```bash
# Spinnaker 用のサービスアカウントの作成
$ gcloud iam service-accounts create spinnaker-account \
    --display-name spinnaker-account

# サービスアカウントのメールアドレスを環境変数に格納
$ export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')
# 取得できるメールアドレスはこんな「spinnaker-account@qwiklabs-gcp-00-c0fdab7fa347.iam.gserviceaccount.com」感じ

# サービスアカウントに Cloud Storage の権限を付与
$ gcloud projects add-iam-policy-binding $PROJECT \
    --role roles/storage.admin \
    --member serviceAccount:$SA_EMAIL

# サービスアカウントキーのダウンロード
$ gcloud iam service-accounts keys create spinnaker-sa.json \
     --iam-account $SA_EMAIL
```

Container Registry からの通知に使用する Cloud Pub/Sub トピックを作成する。

```bash
$ gcloud pubsub topics create projects/$PROJECT/topics/gcr
```

イメージの push についての通知を受け取れるように、Spinnaker から読み取ることができるサブスクリプションを作成する。

```bash
$ gcloud pubsub subscriptions create gcr-triggers \
    --topic projects/${PROJECT}/topics/gcr
```

Spinnaker のサービス アカウントに、gcr-triggers サブスクリプションからの読み取り権限を付与する。

```bash
$ export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')

$ gcloud beta pubsub subscriptions add-iam-policy-binding gcr-triggers \
    --role roles/pubsub.subscriber --member serviceAccount:$SA_EMAIL
```

Spinnaker をデプロイするために Helm を準備する。

```bash
# helm バイナリ取得・展開
$ wget https://get.helm.sh/helm-v3.1.1-linux-amd64.tar.gz
$ tar zxfv helm-v3.1.1-linux-amd64.tar.gz
$ cp linux-amd64/helm .

# Helm にクラスタの cluster-admin ロールを付与
$ kubectl create clusterrolebinding user-admin-binding \
    --clusterrole=cluster-admin --user=$(gcloud config get-value account)

# Spinnaker に cluster-admin ロールを付与し、すべての名前空間にリソースをデプロイできるようにする
$ kubectl create clusterrolebinding --clusterrole=cluster-admin \
    --serviceaccount=default:default spinnaker-admin

# Helm の使用可能なリポジトリに stable チャートのデプロイを追加（Spinnaker も含まれている）
$ ./helm repo add stable https://charts.helm.sh/stable
$ ./helm repo update
```

Spinnaker を作成する。

```bash
# 環境変数の準備
$ export PROJECT=$(gcloud info \
    --format='value(config.project)')
$ export BUCKET=$PROJECT-spinnaker-config

# Cloud Storage バケットの作成
$ gsutil mb -c regional -l us-central1 gs://$BUCKET
```

`spinnaker-config.yaml` ファイルの作成。

```bash
$ export SA_JSON=$(cat spinnaker-sa.json)
$ export PROJECT=$(gcloud info --format='value(config.project)')
$ export BUCKET=$PROJECT-spinnaker-config
$ cat > spinnaker-config.yaml <<EOF
gcs:
  enabled: true
  bucket: $BUCKET
  project: $PROJECT
  jsonKey: '$SA_JSON'

dockerRegistries:
- name: gcr
  address: https://gcr.io
  username: _json_key
  password: '$SA_JSON'
  email: 1234@5678.com

# デフォルトのストレージ バックエンドである minio を無効にします
minio:
  enabled: false

# GCP サービスを有効にするように Spinnaker を構成します
halyard:
  spinnakerVersion: 1.19.4
  image:
    repository: us-docker.pkg.dev/spinnaker-community/docker/halyard
    tag: 1.32.0
    pullSecrets: []
  additionalScripts:
    create: true
    data:
      enable_gcs_artifacts.sh: |-
        \$HAL_COMMAND config artifact gcs account add gcs-$PROJECT --json-path /opt/gcs/key.json
        \$HAL_COMMAND config artifact gcs enable
      enable_pubsub_triggers.sh: |-
        \$HAL_COMMAND config pubsub google enable
        \$HAL_COMMAND config pubsub google subscription add gcr-triggers \
          --subscription-name gcr-triggers \
          --json-path /opt/gcs/key.json \
          --project $PROJECT \
          --message-format GCR
EOF
```

Spinnaker チャートをデプロイする。

```bash
$ ./helm install -n default cd stable/spinnaker -f spinnaker-config.yaml \
           --version 2.0.0-rc9 --timeout 10m0s --wait
```

ポートフォワードして Spinnaker の管理画面を確認する。

```bash
$ export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \
    -o jsonpath="{.items[0].metadata.name}")
$ kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &
```

Cloud Shell ウィンドウの最上部で [ウェブでプレビュー] をクリックし、[ポート 8080 でプレビュー] をクリックすると Spinnaker の管理画面が見れる。  
次に Docker イメージを作成する。

```bash
# サンプルアプリの取得・展開
$ gsutil -m cp -r gs://spls/gsp114/sample-app.tar .
$ mkdir sample-app
$ tar xvf sample-app.tar -C ./sample-app
$ cd sample-app

# git リポジトリの作成
$ gcloud source repos create sample-app

# git 用アカウント情報設定・初回コミット・Push
$ git config --global user.email "$(gcloud config get-value core/account)"
$ git config --global user.name "[USERNAME]"
$ export PROJECT=$(gcloud info --format='value(config.project)')
$ git init
$ git add .
$ git commit -m "Initial commit"
$ git config credential.helper gcloud.sh
$ git remote add origin https://source.developers.google.com/p/$PROJECT/r/sample-app
$ git push origin master
```

次にビルドトリガーを構成する。  
Git タグが リポジトリに push されるたびに Docker イメージをビルドして push するように Container Builder を構成する。  
Container Builder はソースコードを自動的にチェックアウトして、リポジトリ内の Dockerfile から Docker イメージをビルドし、そのイメージを Google Cloud Container Registry に push する。

- Cloud Platform Console で、ナビゲーション メニュー > [Cloud Build] > [トリガー] をクリック
- [トリガーを作成] をクリック
- 次のトリガー設定を指定
    - 名前: sample-app-tags
    - イベント: 新しいタグを push する
    - 新しく作成した sample-app リポジトリを選択
    - タグ: v1.*
    - ビルド構成: Cloud Build 構成ファイル（yaml または json）
    - Cloud Build 構成ファイルの場所: /cloudbuild.yaml

これ以降、文字 "v" が先頭に付いている Git タグをソースコード リポジトリに push すると、Container Builder が自動的にアプリケーションをビルドして、Docker イメージとして Container Registry に push する。

Spinnaker で使用するために Kubernetes マニフェストを準備する。

```bash
$ export PROJECT=$(gcloud info --format='value(config.project)')
$ gsutil mb -l us-central1 gs://$PROJECT-kubernetes-manifests
$ gsutil versioning set on gs://$PROJECT-kubernetes-manifests
$ sed -i s/PROJECT/$PROJECT/g k8s/deployments/*
$ git commit -a -m "Set project ID"
```

Git タグを作成してイメージをビルドしてみる。

```bash
$ git tag v1.0.0
$ git push --tags
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 2 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (8/8), 778 bytes | 778.00 KiB/s, done.
Total 8 (delta 5), reused 0 (delta 0)
remote: Resolving deltas: 100% (5/5)
To https://source.developers.google.com/p/qwiklabs-gcp-00-c0fdab7fa347/r/sample-app
 * [new tag]         v1.0.0 -> v1.0.0
```

Spinnaker を管理する spin CLI をインストールする。

```bash
$ curl -LO https://storage.googleapis.com/spinnaker-artifacts/spin/1.14.0/linux/amd64/spin
$ chmod +x spin
```

spin を使用して、Spinnaker 内に sample という名前のアプリを作成。

```bash
$ ./spin application save --application-name sample \
                        --owner-email "$(gcloud config get-value core/account)" \
                        --cloud-providers kubernetes \
                        --gate-endpoint http://localhost:8080/gate
```

sample-app ソースコード ディレクトリから次のコマンドを実行します。これにより、サンプル パイプラインが Spinnaker インスタンスにアップロードされます。

```bash
$ export PROJECT=$(gcloud info --format='value(config.project)')
$ sed s/PROJECT/$PROJECT/g spinnaker/pipeline-deploy.json > pipeline.json
$ ./spin pipeline save --gate-endpoint http://localhost:8080/gate -f pipeline.json
```

パイプラインの実行を手動でトリガーして確認する。

- Spinnaker UI で、画面の上部にある [Applications] -> sampleをクリック。
- 上部にある [パイプライン] をクリックして、アプリケーションのパイプラインのステータスを表示。
- パイプラインを初めてトリガーするために、[Start Manual Execution] をクリック -> RUN 。
- [Execution Details] をクリックして、パイプラインの進行状況の詳細を表示。
- 進行状況バーで、デプロイメント パイプラインのステータスとそのステップを確認できる。
- ステージをクリックして、その詳細を表示します。
- 3～5 分後に統合テストフェーズが完了すると、パイプラインがデプロイメントの続行を手動で承認するよう求めてくる。
- 黄色の「人物」アイコンにカーソルを合わせ、[Continue] をクリック。
    - ロールアウトが本番環境フロントエンドと本番環境バックエンドのデプロイメントに進みます。これは数分後に完了します。
- Spinnaker UI の上部で [Infrastructure] > [Load Balancers] をクリックして、アプリを表示。
- ロードバランサのリストを下にスクロールし、[service sample-frontend-production] の下の [Default] をクリックするとロードバランサの詳細がページの右側に表示される。
- 右側の詳細ペインをスクロール ダウンし、Ingress IP のクリップボード ボタンをクリックしてアプリの IP アドレスをコピーする。
- コピーしたアドレスを新しいブラウザタブに貼り付けて、アプリケーションを表示
    - canary バージョンが表示される場合があるが、画面を更新すると production バージョンも表示される
- アプリケーションをビルド、テスト、デプロイするためのパイプラインを手動でトリガーした。

次に、コードの変更によるパイプラインのトリガーを行う。

```bash
# sample-app ディレクトリから次のコマンドを実行して、アプリの色をオレンジ色から青色に変更
$ sed -i 's/orange/blue/g' cmd/gke-info/common-service.go

# 変更にタグを付け、ソースコード リポジトリに push
$ git commit -a -m "Change color to blue"
$ git tag v1.0.1
$ git push --tags
```

- GCP Console で [Cloud Build] > [履歴] を開き、新しいビルドが表示されるまで数分待つ。（必要であればページを更新）  
- 新しいビルドが完了するまで待ってから次のステップに進む。
- Spinnaker UI に戻り、[Pipelines] をクリックして、イメージのデプロイを開始するパイプラインの動作を監視する。
    - 自動的にトリガーされたパイプラインは、表示されるまでに数分かかる。（必要であればページを更新）

次に、カナリア デプロイメントを観察する。

- デプロイが一時停止し、本番環境へのロールアウトを待機している間に、実行中のアプリケーションを表示しているウェブページに戻る。
    - アプリが表示されているタブを繰り返し更新。バックエンドのうちの 4 つがアプリの以前のバージョンを実行しており、カナリアを実行しているバックエンドは 1 つだけ。
    - 5 回更新するごとにアプリの新しい青色バージョンが表示されるはず。
- パイプラインが完了すると、アプリは以下のようになる。（コードを変更したため、色は青色。また、[バージョン] フィールドには canary ）
    - Backend that serviced this request
        - Pod Name: sample-backend-canary-xxxx
        - Node Name: gke-spinnaker-tutorial-default-pool-xxxx
        - Version: Canary
        - Zone: project/xxxx/zones/us-central1-f
        - Project: qwiklabs-gcp-00-xxxx
        - Node Internal IP: 10.128.0.4
        - Node External IP: 146.148.80.155
    - Frontend that handled this request
        - Frontend IP:PORT: 10.40.2.14.48884
        - Request to Backend: GET /metadata....
        - Error: None
- 必要に応じて前回の commit を元に戻し、この変更をロールバックできる。ロールバックすると新しいタグ　(v1.0.2）が追加され、v1.0.1 をデプロイするときに使用したのと同じパイプラインを通してタグが戻される。
    - `git revert v1.0.1`
    - Ctrl + O、ENTER、CTRL + X を押す
    - `git tag v1.0.2`
    - `git push --tags`
- ビルドとパイプラインが完了したら、ロールバックを確認する
    - [INFRASTRUCTURE] > [LOAD BALANCERS] の順にクリックしてから、[service sample-frontend-production] の [DEFAULT] をクリックして Ingress IP アドレスをコピー。このアドレスを新しいタブに貼り付け。
    - アプリがオレンジ色に戻り、production バージョン番号を確認できる。

### 3.5. Cloud Monitoring APM によるサイトの信頼性のトラブルシューティング

Monitoring ワークスペースを作成して監視を行う。

- Cloud Console で、ナビゲーション メニュー > [モニタリング] をクリック
- ワークスペースがプロビジョニングされるまで待つ

次に、SLO と SLI を作成する。

- サービスレベル指標（SLI）
    - 例）リクエストのレイテンシ（リクエストに対してレスポンスを返すのにかかる時間）、エラー率、システム スループット（一般的には 1 秒あたりのリクエスト数）、可用性（サービスが使用可能な時間の割合）、耐久性（データが長期間にわたって保持される可能性）
- サービスレベル目標（SLO）
- サービスレベル契約（SLA）

例えば、SLO、SLI は以下のようなものである。

|SLI|指標|説明|SLO|
|---|---|---|---|
|リクエストのレイテンシ|フロントエンドのレイテンシ|ユーザーがページの読み込みを待機している時間。|過去 60 分間で 99% のリクエストが 3 秒以内に処理されている|
|エラー率|フロントエンドのエラー率|ユーザーが経験したエラー率。|過去 60 分間のエラー数が 0|
|エラー率|チェックアウトのエラー率|チェックアウト サービスを呼び出す他のサービスが経験したエラー率。|過去 60 分間のエラー数が 0|
|エラー率|通貨サービスのエラー率|通貨サービスを呼び出す他のサービスが経験したエラー率。|過去 60 分間のエラー数が 0|
|可用性|フロントエンドの成功率|サービスの可用性の判断基準として、成功したリクエストの割合。|過去 60 分間で 99% のリクエストが成功している|

Cloud Platform Console の Cloud モニタリング（ナビゲーション メニュー > [モニタリング]）で、左側のメニューにある [アラート]、[CREATE POLICY] の順にクリックし、SLO を監視するアラートを作成する。