---
title: "CLI"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 3
---

1. gcloud

<!--more-->

## 1. gcloud

- [gcloud コマンドライン ツールの概要](https://cloud.google.com/sdk/gcloud)
- [gcloud リファレンス](https://cloud.google.com/sdk/gcloud/reference/?hl=ja)

### 1.1. インストール

Mac へのインストールは以下。

```zsh
% asdf plugin add gcloud
✅  All dependencies found on system!

% asdf list-all gcloud
334.0.0

% asdf install gcloud 334.0.0
⏬  downloading google-cloud-sdk-334.0.0-darwin-x86_64.tar.gz
########################################################################################################################################### 100.0%
✅  downloaded!
✅  All dependencies found on system!
📦  extracting...
✅  extracted!
🚧  installing...
Welcome to the Google Cloud SDK!

Your current Cloud SDK version is: 334.0.0
The latest available version is: 334.0.0

┌────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                 Components                                                 │
├───────────────┬──────────────────────────────────────────────────────┬──────────────────────────┬──────────┤
│     Status    │                         Name                         │            ID            │   Size   │
├───────────────┼──────────────────────────────────────────────────────┼──────────────────────────┼──────────┤
│ Not Installed │ App Engine Go Extensions                             │ app-engine-go            │  4.8 MiB │
│ Not Installed │ Appctl                                               │ appctl                   │ 18.5 MiB │
│ Not Installed │ Cloud Bigtable Command Line Tool                     │ cbt                      │  7.6 MiB │
│ Not Installed │ Cloud Bigtable Emulator                              │ bigtable                 │  6.6 MiB │
│ Not Installed │ Cloud Datalab Command Line Tool                      │ datalab                  │  < 1 MiB │
│ Not Installed │ Cloud Datastore Emulator                             │ cloud-datastore-emulator │ 18.4 MiB │
│ Not Installed │ Cloud Firestore Emulator                             │ cloud-firestore-emulator │ 41.6 MiB │
│ Not Installed │ Cloud Pub/Sub Emulator                               │ pubsub-emulator          │ 60.4 MiB │
│ Not Installed │ Cloud SQL Proxy                                      │ cloud_sql_proxy          │  7.4 MiB │
│ Not Installed │ Emulator Reverse Proxy                               │ emulator-reverse-proxy   │ 14.5 MiB │
│ Not Installed │ Google Cloud Build Local Builder                     │ cloud-build-local        │  6.2 MiB │
│ Not Installed │ Google Container Registry's Docker credential helper │ docker-credential-gcr    │  2.2 MiB │
│ Not Installed │ Kustomize                                            │ kustomize                │ 22.8 MiB │
│ Not Installed │ Minikube                                             │ minikube                 │ 23.7 MiB │
│ Not Installed │ Nomos CLI                                            │ nomos                    │ 19.8 MiB │
│ Not Installed │ On-Demand Scanning API extraction helper             │ local-extract            │ 11.5 MiB │
│ Not Installed │ Skaffold                                             │ skaffold                 │ 17.5 MiB │
│ Not Installed │ anthos-auth                                          │ anthos-auth              │ 16.3 MiB │
│ Not Installed │ config-connector                                     │ config-connector         │ 44.1 MiB │
│ Not Installed │ gcloud Alpha Commands                                │ alpha                    │  < 1 MiB │
│ Not Installed │ gcloud Beta Commands                                 │ beta                     │  < 1 MiB │
│ Not Installed │ gcloud app Java Extensions                           │ app-engine-java          │ 53.1 MiB │
│ Not Installed │ gcloud app PHP Extensions                            │ app-engine-php           │ 21.9 MiB │
│ Not Installed │ gcloud app Python Extensions                         │ app-engine-python        │  6.1 MiB │
│ Not Installed │ gcloud app Python Extensions (Extra Libraries)       │ app-engine-python-extras │ 27.1 MiB │
│ Not Installed │ kpt                                                  │ kpt                      │ 13.1 MiB │
│ Not Installed │ kubectl                                              │ kubectl                  │  < 1 MiB │
│ Not Installed │ kubectl-oidc                                         │ kubectl-oidc             │ 16.3 MiB │
│ Not Installed │ pkg                                                  │ pkg                      │          │
│ Installed     │ BigQuery Command Line Tool                           │ bq                       │  < 1 MiB │
│ Installed     │ Cloud SDK Core Libraries                             │ core                     │ 17.8 MiB │
│ Installed     │ Cloud Storage Command Line Tool                      │ gsutil                   │  3.9 MiB │
└───────────────┴──────────────────────────────────────────────────────┴──────────────────────────┴──────────┘
To install or remove components at your current SDK version [334.0.0], run:
  $ gcloud components install COMPONENT_ID
  $ gcloud components remove COMPONENT_ID

To update your SDK installation to the latest version [334.0.0], run:
  $ gcloud components update

% asdf global gcloud 334.0.0

% asdf current gcloud
gcloud          334.0.0         /Users/xxxxx/.tool-versions

% gcloud version
Google Cloud SDK 334.0.0
bq 2.0.66
core 2021.03.26
gsutil 4.60
```

### 1.2. 環境・構成の管理

アカウント情報系の設定？ -> https://qiita.com/sonots/items/906798c408132e26b41c

#### 1.2.1 Google Cloud SDK

```bash
# Google Cloud SDK のプロパティを見る
$ gcloud config list

# すべてのプロパティを見る
$ gcloud config list --all

# Google Cloud SDK のコンポーネントのリストを取得
$ gcloud components list

Your current Cloud SDK version is: 334.0.0
The latest available version is: 335.0.0
┌───────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                   Components                                                  │
├──────────────────┬──────────────────────────────────────────────────────┬──────────────────────────┬──────────┤
│      Status      │                         Name                         │            ID            │   Size   │
├──────────────────┼──────────────────────────────────────────────────────┼──────────────────────────┼──────────┤
│ Update Available │ Cloud SDK Core Libraries                             │ core                     │ 17.8 MiB │
│ Update Available │ gcloud app Java Extensions                           │ app-engine-java          │ 53.1 MiB │
│ Deprecated       │ Kind                                                 │ kind                     │          │
│ Not Installed    │ Appctl                                               │ appctl                   │ 21.0 MiB │
│ Not Installed    │ Cloud Firestore Emulator                             │ cloud-firestore-emulator │ 41.9 MiB │
│ Not Installed    │ Cloud SQL Proxy                                      │ cloud_sql_proxy          │  7.6 MiB │
│ Not Installed    │ Cloud Spanner Emulator                               │ cloud-spanner-emulator   │ 21.8 MiB │
│ Not Installed    │ Emulator Reverse Proxy                               │ emulator-reverse-proxy   │ 14.5 MiB │
│ Not Installed    │ Google Container Registry's Docker credential helper │ docker-credential-gcr    │  1.8 MiB │
│ Not Installed    │ Kustomize                                            │ kustomize                │ 25.9 MiB │
│ Not Installed    │ Nomos CLI                                            │ nomos                    │ 22.9 MiB │
│ Not Installed    │ anthos-auth                                          │ anthos-auth              │ 16.4 MiB │
│ Not Installed    │ config-connector                                     │ config-connector         │ 43.6 MiB │
│ Not Installed    │ kubectl                                              │ kubectl                  │  < 1 MiB │
│ Not Installed    │ kubectl-oidc                                         │ kubectl-oidc             │ 16.4 MiB │
│ Not Installed    │ pkg                                                  │ pkg                      │          │
│ Installed        │ App Engine Go Extensions                             │ app-engine-go            │  4.9 MiB │
│ Installed        │ BigQuery Command Line Tool                           │ bq                       │  < 1 MiB │
│ Installed        │ Cloud Bigtable Command Line Tool                     │ cbt                      │  7.7 MiB │
│ Installed        │ Cloud Bigtable Emulator                              │ bigtable                 │  6.6 MiB │
│ Installed        │ Cloud Datalab Command Line Tool                      │ datalab                  │  < 1 MiB │
│ Installed        │ Cloud Datastore Emulator                             │ cloud-datastore-emulator │ 18.4 MiB │
│ Installed        │ Cloud Pub/Sub Emulator                               │ pubsub-emulator          │ 60.4 MiB │
│ Installed        │ Cloud Storage Command Line Tool                      │ gsutil                   │  3.9 MiB │
│ Installed        │ Google Cloud Build Local Builder                     │ cloud-build-local        │  6.3 MiB │
│ Installed        │ Minikube                                             │ minikube                 │ 23.9 MiB │
│ Installed        │ On-Demand Scanning API extraction helper             │ local-extract            │ 13.5 MiB │
│ Installed        │ Skaffold                                             │ skaffold                 │ 16.6 MiB │
│ Installed        │ gcloud Alpha Commands                                │ alpha                    │  < 1 MiB │
│ Installed        │ gcloud Beta Commands                                 │ beta                     │  < 1 MiB │
│ Installed        │ gcloud app Python Extensions                         │ app-engine-python        │  6.1 MiB │
│ Installed        │ gcloud app Python Extensions (Extra Libraries)       │ app-engine-python-extras │ 27.1 MiB │
│ Installed        │ kpt                                                  │ kpt                      │ 12.5 MiB │
└──────────────────┴──────────────────────────────────────────────────────┴──────────────────────────┴──────────┘
To install or remove components at your current SDK version [334.0.0], run:
  $ gcloud components install COMPONENT_ID
  $ gcloud components remove COMPONENT_ID
To update your SDK installation to the latest version [335.0.0], run:
  $ gcloud components update
```

[コンポーネント](https://cloud.google.com/sdk/docs/components) は、個別にインストール可能な SDK の構成要素で、マンドライン ツール（gcloud、bq、gsutil）、アルファ版 / ベータ版リリースレベルの gcloud CLI コマンドや、SDK の特定のツールとの依存関係を含むパッケージなどがある。  
以下のようなコマンドで管理できる。

```bash
# コンポーネント一覧
$ gcloud components list

# コンポーネントのインストール
$ gcloud components install COMPONENT_ID
# インストール例
$ gcloud components install kubectl

# コンポーネントの更新
$ gcloud components update
# 指定バージョンへの変更
$ gcloud components update --version VERSION

# コンポーネントの削除
$ gcloud components remove COMPONENT_ID
```

#### 1.2.2. プロジェクト

```bash
# プロジェクト一覧
$ gcloud projects list

# プロジェクト切り替え
$ gcloud config set project <your-project-id>
```

#### 1.2.3. リージョン・ゾーン

リージョン・ゾーンなどの設定情報の取得は以下。

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

### 1.3. 使い方

```zsh
gcloud GROUP | COMMAND [--account=ACCOUNT]
    [--billing-project=BILLING_PROJECT] [--configuration=CONFIGURATION]
    [--flags-file=YAML_FILE] [--flatten=[KEY,...]] [--format=FORMAT]
    [--help] [--project=PROJECT_ID] [--quiet, -q]
    [--verbosity=VERBOSITY; default="warning"] [--version, -v] [-h]
    [--impersonate-service-account=SERVICE_ACCOUNT_EMAILS] [--log-http]
    [--trace-token=TRACE_TOKEN] [--no-user-output-enabled]
````

GROUP は `gcloud` コマンドで管理できるリソースを指し、以下がある。  
ただし、 GROUP は階層的になっており、以下から数段続く場合がある。

|GROUP|description|
|:---|:---|
|access-context-manager|Manage Access Context Manager resources.|
|active-directory|Manage Managed Microsoft AD resources.|
|ai|Manage entities in AI Platform.|
|ai-platform|Manage AI Platform jobs and models.|
|alpha|(ALPHA) Alpha versions of gcloud commands.|
|anthos|Anthos command Group.|
|api-gateway|Manage Cloud API Gateway resources.|
|apigee|Manage Apigee resources.|
|app|Manage your App Engine deployments.|
|artifacts|Manage Artifact Registry resources.|
|asset|Manage the Cloud Asset Inventory.|
|assured|Read and manipulate Assured Workloads data controls.|
|auth|Manage oauth2 credentials for the Google Cloud SDK.|
|beta|(BETA) Beta versions of gcloud commands.|
|bigtable|Manage your Cloud Bigtable storage.|
|billing|Manage billing accounts and associate them with projects.|
|builds|Create and manage builds for Google Cloud Build.|
|cloud-shell|Manage Google Cloud Shell.|
|components|List, install, update, or remove Google Cloud SDK components.|
|composer|Create and manage Cloud Composer Environments.|
|compute|Create and manipulate Compute Engine resources.|
|config|View and edit Cloud SDK properties.|
|container|Deploy and manage clusters of machines for running containers.|
|data-catalog|Manage Cloud Data Catalog resources.|
|database-migration|Manage Database Migration Service resources.|
|dataflow|Manage Google Cloud Dataflow resources.|
|dataproc|Create and manage Google Cloud Dataproc clusters and jobs.|
|datastore|Manage your Cloud Datastore resources.|
|debug|Commands for interacting with the Cloud Debugger.|
|deployment-manager|Manage deployments of cloud resources.|
|dns|Manage your Cloud DNS managed-zones and record-sets.|
|domains|Manage domains for your Google Cloud projects.|
|emulators|Set up your local development environment using emulators.|
|endpoints|Create, enable and manage API services.|
|eventarc|Manage Eventarc resources.|
|filestore|Create and manipulate Cloud Filestore resources.|
|firebase|Work with Google Firebase.|
|firestore|Manage your Cloud Firestore resources.|
|functions|Manage Google Cloud Functions.|
|game|Managed Cloud Game Services.|
|healthcare|Manage Cloud Healthcare resources.|
|iam|Manage IAM service accounts and keys.|
|iap|Manage IAP policies.|
|identity|Manage Cloud Identity Groups and Memberships resources.|
|iot|Manage Cloud IoT resources.|
|kms|Manage cryptographic keys in the cloud.|
|logging|Manage Cloud Logging.|
|memcache|Manage Cloud Memorystore Memcached resources.|
|metastore|Manage Dataproc Metastore resources.|
|ml|Use Google Cloud machine learning capabilities.|
|ml-engine|Manage AI Platform jobs and models.|
|monitoring|Manage Cloud Monitoring dashboards.|
|network-management|Manage Network Management resources.|
|notebooks|Notebooks Command Group.|
|org-policies|Create and manage Organization Policies.|
|organizations|Create and manage Google Cloud Platform Organizations.|
|policy-troubleshoot|Troubleshoot Google Cloud Platform policies.|
|projects|Create and manage project access policies.|
|pubsub|Manage Cloud Pub/Sub topics, subscriptions, and snapshots.|
|recommender|Manage Cloud recommendations and recommendation rules.|
|redis|Manage Cloud Memorystore Redis resources.|
|resource-manager|Manage Cloud Resources.|
|run|Manage your Cloud Run applications.|
|scc|Manage Cloud SCC resources.|
|scheduler|Manage Cloud Scheduler jobs and schedules.|
|secrets|Manage secrets on Google Cloud.|
|service-directory|Command groups for Service Directory.|
|services|List, enable and disable APIs and services.|
|source|Cloud git repository commands.|
|spanner|Command groups for Cloud Spanner.|
|sql|Create and manage Google Cloud SQL databases.|
|tasks|Manage Cloud Tasks queues and tasks.|
|topic|gcloud supplementary help.|
|workflows|Manage your Cloud Workflows resources.|
|workspace-add-ons|Manage Google Workspace Add-ons resources.|

COMMAND は GROUP によって異なるが、共通的なものに以下がある。

|COMMAND|description|
|:---|:---|
|list|リソース一覧を表示|
|describe|指定したリソースの詳細を表示|
|create|指定したリソースを作成|
|update|指定したリソースを更新|
|delete|指定したリソースを削除|