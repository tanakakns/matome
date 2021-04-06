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

### 1.1. 設定

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

### 1.2. 使い方

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