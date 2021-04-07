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

（教材メモ：https://www.qwiklabs.com/focuses/3563?parent=catalog）

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

## 3. Google Kubernetes Engine

1. [コンセプト](#3-1-コンセプト)
2. [クラスタ作成](#3-2-クラスタ作成)

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