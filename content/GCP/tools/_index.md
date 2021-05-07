---
title: "ツール"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 2
---

Cloud Console、Cloud Shell、API とサービス、Google Cloud SDK、Cloud Deployment Manager

<!--more-->

## 1. Cloud Console

- [Cloud Console](https://cloud.google.com/cloud-console/)は、Google Cloud の開発ツールの中核であるウェブコンソール。
- ログインするにはユーザー名とパスワードが必要で、[Cloud Identity and Access Management（Cloud IAM）](https://cloud.google.com/iam/) などの機能によりアクセス制御される。

## 2. Cloud Shell

- [Cloud Shell](https://cloud.google.com/shell/docs/features) はブラウザで機能するコマンド実行環境。
- その実態は Debian ベースの仮想マシンで動作するターミナルで、Unix の標準コマンドや、nano などのテキストエディタの他にも様々な開発ツール（ Google Cloud SDK や git など）、永続的な 5 GB のホーム ディレクトリが用意されている。
- Cloud Shell を使用すると、Cloud Console を離れずに認証済みで最新のシェルコマンドを実行できる。
- なお、Cloud Shell ホームディレクトリ（ `$HOME` ）の内容は、複数のプロジェクトおよびすべての Cloud Shell セッション間で保持され、仮想マシンを終了して再起動した後でも消えずに残る。

## 3. API とサービス

Google Cloud API は、ビジネス管理から機械学習にまで及ぶさまざまな分野の API が 200 以上用意されており、サービスと同様に、すべて Google Cloud プロジェクト / アプリケーションと簡単に統合できる。  
[API 設計ガイド](https://cloud.google.com/apis/design/) で説明されているリソース指向の設計原則が適用されている。  
なお、 Cloud API を利用する際は、各自で有効に設定する必要がある。（サイドメニュー「API とサービス」 -> 「ライブラリ」から APIを検索して「有効にする」）  
以下のコマンドでも API の有効化が可能。

```bash
# API 有効化
$ gloucd services enable SERVICE_NAME

# 現在のプロジェクトで有効になっているAPIの一覧を表示
$ gcloud services list

# 現在のプロジェクトで有効にできるサービスの一覧を表示
$ gcloud services list --available
```

なお、アカウントに対して、いかにロールで権限を与えようが、 **API を有効化しないと利用できない** ことに注意。

- [Google APIs Explorer](https://developers.google.com/apis-explorer/#p/)

## 4. Google Cloud SDK

[Google Cloud SDK](https://cloud.google.com/sdk/docs) は、GCP にホスティングされているリソースとアプリケーションの管理に使用できる一連のツール。

- `gcloud` CLI： Google Cloud APIs の操作をコマンドラインで実施できる
- クライアントライブラリ：各プログラミング言語のクライアントライブラリ
- プロダクト固有のコマンドラインツール
    - `gsutil` ： Cloud Storage
    - `bq` ： BigQuery
    - `kubectl` ： k8s
    - など

### 4.1. gcloud

- [gcloud コマンドライン ツールの概要](https://cloud.google.com/sdk/gcloud)
- [gcloud リファレンス](https://cloud.google.com/sdk/gcloud/reference/?hl=ja)

#### 4.1.1. インストール

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

#### 4.1.2. 使い方

```bash
$ gcloud GROUP COMMAND OPTIONS
```

GROUP は多段になっており、  `gcloud compute images list` （ `compute image` が GROUP で `list` が COMMAND ）のように 2 コ以上続くこともある。  
COMMAND は GROUP によって異なるが、共通的なものに以下がある。

|COMMAND|description|
|:---|:---|
|list|リソース一覧を表示|
|describe|指定したリソースの詳細を表示|
|create|指定したリソースを作成|
|update|指定したリソースを更新|
|delete|指定したリソースを削除|

たとえば、有効なアカウント名の一覧を取得するコマンドは以下。（ GROUP が 1 段の例）

```bash
$ gcloud auth list

Credentialed Accounts
ACTIVE  ACCOUNT
*       student-00-918550b6c355@qwiklabs.net
To set the active account, run:
    $ gcloud config set account `ACCOUNT`
```

たとえば、プロジェクト ID の一覧を取得するコマンドは以下。（ GROUP が 2 段の例）

```bash
$ gcloud config list project

[core]
project = qwiklabs-gcp-44776a13dea667a6
```

#### 4.1.3. アカウント情報

アカウント情報系の設定？ -> https://qiita.com/sonots/items/906798c408132e26b41c

#### 4.1.4. 構成・設定の管理

`gcloud` コマンドは他の Google Cloud SDK のツールである `bq` や `gsutil` などの設定情報も管理する。  
Google Cloud SDK に含まれる `bq` や `gsutil` などの構成要素を [コンポーネント](https://cloud.google.com/sdk/docs/components) という。  
つまり `gcloud` は Google Cloud SDK のコンポーネントを管理する機能も含むことになる。  
Google Cloud SDK の設定情報の管理は以下のコマンドを利用する。


```bash
# Google Cloud SDK の設定一覧表示
$ gcloud config list

# 未設定の設定も含めて全て表示
$ gcloud config list --all

# コンポーネントの一覧を表示（インストール時に表示されたどでかいテーブルが出る）
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

#### 4.1.5. よく使うコマンド

プロジェクト情報の管理。

```bash
# プロジェクト一覧
$ gcloud projects list

# プロジェクト切り替え
$ gcloud config set project <your-project-id>
```

リージョン・ゾーンなどの設定情報の管理。

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

デフォルトのリージョン・ゾーンなどの情報の設定。

```bash
# デフォルトのリージョン設定
$ gcloud config set compute/region <REGION>

# デフォルトのゾーン設定（なお、ゾーンにリージョン情報も含まれるため設定せずとも推測される場合もある）
$ gcloud config set compute/zone <ZONE>
```

## 5. Cloud Deployment Manager

AWS でいう CloudFormation 。  
単一の API に対応した **リソース** とそのリソースをまとめた **デプロイメント** などで構成される。  
`gcloud deployment-manager deployments` コマンドでデプロイメントを管理し、 `gcloud deployment-manager resouces` コマンドでデプロイメントに含まれるリソースを管理する。