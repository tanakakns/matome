---
title: "入門"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

1. [コンセプト](#1-コンセプト)
2. [ツール](#2-ツール)
3. [リソース階層とアクセス制御](#3-リソース階層とアクセス制御)
4. リソースの管理
5. 監視
6. コスト管理
7. [Google Cloud サービス](#4-google-cloud-サービス)

<!--more-->

## 1. コンセプト

- [リージョンとゾーン](https://cloud.google.com/compute/docs/regions-zones/) ※[地域とリージョン](https://cloud.google.com/docs/geography-and-regions?hl=ja)
    - 一部の Compute Engine リソースは、リージョン内またはゾーン内にのみ存在する
    - **リージョン** とはリソースを実行できる特定の地理的な場所で、各リージョンには、1 つまたは複数の **ゾーン**がある
        - たとえば、リージョン `us-central1` は米国中部を指し、ゾーン `us-central1-a`、`us-central1-b`、`us-central1-c`、`us-central1-f` が含まれる
    - ゾーンは、リージョン内の単一の障害ドメインとみなすことができる
        -  GCP のゾーンは、 AWS でいう AZ：アベイラビリティゾーン
- リソース
    - リソースは大きく、 **グローバルリソース** （マルチリージョンリソース）、 **リージョンリソース** 、 **ゾーンリソース** の 3 つのスコープに分類され、アクセス可能なリソースが決まる（ [詳細](https://cloud.google.com/compute/docs/regions-zones/global-regional-zonal-resources?hl=ja) ）
    - グローバルリソースには同じプロジェクトにある任意のリージョンまたはゾーンのリソースからアクセスできる
    - リージョンリソースにアクセスできるのは、同じリージョン内のリソースだけ
        - インスタンスに静的 IP アドレスを割り当てるには、インスタンスが静的 IP と同じリージョンに存在している必要がある
    - ゾーンリソースにアクセスできるのは、同じゾーン内のリソースだけ
        - 永続ディスクをインスタンスに接続するには、両リソースを同ゾーンに配置する必要がある
- **プロジェクト**
    - リソースと課金を管理する単位
        - 請求先アカウントをリンクすることで課金が有効
        - [Pricing Calculator](https://cloud.google.com/products/calculator?hl=ja)（コスト見積のための Web UI）
    - すべての GCP リソースは 1 つのプロジェクトに関連付けられる
    - 各種設定と権限も含まれており、これを使用して、セキュリティルールとどのアカウントがどのリソースにアクセスできるかが指定される
        - [共有 VPC](https://cloud.google.com/vpc/docs/shared-vpc?hl=ja) または [VPC ネットワークピアリング](https://cloud.google.com/vpc/docs/vpc-peering?hl=ja)を使用しない限り、あるプロジェクトが別のプロジェクトのリソースにアクセスすることはできない
    - 複数のプロジェクトを束ねて管理するには [組織](https://cloud.google.com/resource-manager/docs/creating-managing-organization?hl=ja) を使う
    - プロジェクトは **プロジェクト名** （任意のプロジェクト表示名）、 **プロジェクトID** （カスタマイズ可能な識別子）、 **プロジェクト番号** （GCPから割り当てられる識別子）からなる

## 2. ツール

### 2.1. Cloud Console

[Cloud Console](https://cloud.google.com/cloud-console/)は、Google Cloud の開発ツールの中核であるウェブコンソール。  
ログインするにはユーザー名とパスワードが必要で、後述の[Cloud Identity and Access Management（Cloud IAM）](https://cloud.google.com/iam/)によりアクセス権限（ロール）管理される。

### 2.2. Google Cloud SDK

[Google Cloud SDK](https://cloud.google.com/sdk/docs) は、GCP にホスティングされているリソースとアプリケーションの管理に使用できる一連のツール。  
以下からなる。

- gcloud CLI： Google Cloud APIs の操作をコマンドラインで実施できる
- クライアントライブラリ：各プログラミング言語のクライアントライブラリ
- プロダクト固有のコマンドラインツール
    - `gsutil` ： Cloud Storage
    - `bq` ： BigQuery
    - `kubectl` ： k8s

#### 2.2.1. gcloud

以下のように利用できる CLI ツール。

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

- [gcloud コマンドライン ツールの概要](https://cloud.google.com/sdk/gcloud)
- [gcloud リファレンス](https://cloud.google.com/sdk/gcloud/reference/?hl=ja)

### 2.3. Cloud Shell

[Cloud Shell](https://cloud.google.com/shell/docs/features) はブラウザで機能するコマンド実行環境。  
その実態は Debian ベースの仮想マシンで、さまざまな開発ツール（gcloud、git など）や、永続的な 5 GB のホーム ディレクトリが用意されている。  
ターミナルにコマンドを入力して、Google Cloud プロジェクトのリソースやサービスを管理できる。  
Cloud Shell を使用すると、Cloud Console を離れずに認証済みで最新のすべてのシェルコマンドを実行できる。  
Google Cloud Console のタイトルバーで、「Cloud Shell をアクティブにする」 をクリックすることで利用できる。  
Cloud Shell には固有のコマンドラインツールがあらかじめ実装されおり、メインの Google Cloud ツールキットは [gcloud](https://cloud.google.com/sdk/gcloud/) で、リソース管理やユーザー認証など、プラットフォーム上のさまざまなタスクに使用される。  
あらかじめインストールされてたツールキットのほかにも、Unix の標準コマンドや、nano などのテキスト エディタも備わっている。

なお、Cloud Shell ホームディレクトリ（ `$HOME` ）の内容は、複数のプロジェクトおよびすべての Cloud Shell セッション間で保持され、仮想マシンを終了して再起動した後でも消えずに残りる。

### 2.4. API とサービス

Google Cloud API は、ビジネス管理から機械学習にまで及ぶさまざまな分野の API が 200 以上用意されており、サービスと同様に、すべて Google Cloud プロジェクト / アプリケーションと簡単に統合できる。  
[API 設計ガイド](https://cloud.google.com/apis/design/) で説明されているリソース指向の設計原則が適用されている。  
なお、 Cloud API を利用する際は、各自で有効に設定する必要がある。（サイドメニュー「API とサービス」 -> 「ライブラリ」から APIを検索して「有効にする」）  
以下のコマンドでも API の有効化が可能。

```bash
# API 有効化
$ gloucd services enable SERVICE_NAME

# 現在のプロジェクトで有効になっているAPIの一覧を表示
gcloud services list
# 現在のプロジェクトで有効にできるサービスの一覧を表示
gcloud services list --available
```

なお、アカウントに対して、いかにロールで権限を与えようが、 **API を有効化しないと利用できない** ことに注意。

- [Google APIs Explorer](https://developers.google.com/apis-explorer/#p/)

## 3. リソース階層とアクセス制御

### 3.1. Cloud Identity and Access Management（Cloud IAM）

[Cloud Identity and Access Management（Cloud IAM）](https://cloud.google.com/iam/docs/how-to?hl=ja) は、「誰が」「どういう操作を」「何に対して」行う権限を持つかを管理するサービス。

|制御内容|具体例|
|:---|:---|
|誰が|メンバー|
|どういう操作を|ロール|
|何に対して|リソース|

Cloud Console の 「サイドメニュー」->「 IAM と管理」->「 IAM 」から利用できる。

### 3.2. メンバー/「誰が」

リソースへのアクセスが許可される主体（「誰が」）には以下がある。

|主体|内容|具体例|
|:---|:---|:---|
|Google アカウント|Google アカウントに関連づけられているメールアドレス| `@gmail.com` やその他ドメインのメールアドレス|
|Google グループ|Google アカウントとサービスアカウントの名前付きコレクション|グループ固有メールアドレス|
|サービスアカウント|個々のエンドユーザではなく、アプリケーションのアカウント| `[サービスアカウント]@[プロジェクトID].iam.gserviceaccount.com` |
|G Suite ドメイン|組織のインターネットドメイン名| `example.com` |
|Cloud Identity ドメイン|組織のインターネットドメイン名（G Suite の機能にはアクセスできない）| `example.com` |

- サービスアカウント
    - アプリケーションやサーバのための ID
    - サービスアカウント事態もリソースで、人が「サービスアカウントとして実行」もできる
    - サービスアカウントによって使われるキーは自動的に管理される
    - ユーザがキーを作成・ダウンロードすることも可能
    - `gcloud iam service-accounts create SERVICE_ACCOUNT_ID` でサービスアカウントを作成する
    - サービスアカウントには **アクセススコープ** を設定できる（以下、例）
        - `https://www.googleapis.com/auth/cloud-platform` ：すべての Google Cloud リソースに対する完全アクセス権
        - `https://www.googleapis.com/auth/compute` ：Compute Engine メソッドに対するフル コントロール アクセス権
        - `https://www.googleapis.com/auth/compute.readonly` ：Compute Engine メソッドに対する読み取り専用アクセス権
        - `https://www.googleapis.com/auth/devstorage.read_only` ：Cloud Storage に対する読み取り専用アクセス権
        - `https://www.googleapis.com/auth/logging.write` ：Compute Engine ログに対する書き込みアクセス権
    - アクセススコープでは完全な権限(/auth/cloud-platform)を付与し、IAMロールで権限を絞るのがおすすめ

### 3.3. 権限とロール/「どういう操作を」

リソースに対する操作の権限は以下の命名規則で表現される。

```zsh
<サービス名>.<リソース>.<動詞>

ex)
compute.instance.delete
compute.instance.start
compute.instance.setMachineType
```

ロールには以下の種類がある。（ [詳細](https://cloud.google.com/iam/docs/understanding-roles/#primitive%5C_roles) ）

|ロール|内容|
|:---|:---|
|基本ロール| **プロジェクト単位** の権限セット。「オーナー」「編集者」「閲覧者」|
|事前定義ロール| **1 サービス単位** の権限セット。「<サービス名>オーナー」「<サービス名>編集者」「<サービス名>閲覧者」|
|カスタムロール|カスタムな権限セット。バケット一覧は取得できないが、特定のバケットには書き込める、とか。|

権限付与には **最小権限の原則** （より細かいロールで必要最低限の権限のみ付与する） を適用すること。  
また、異なる権限を必要とする複数のサービスがある場合は、サービス毎に個別のサービスアカウントを作成し、各サービスアカウントに必要な権限のみ付与すること。  
なお、基本ロールのそれぞれの権限は以下の通り。

|ロール名|権限|
|:---|:---|
|オーナー（ `roles/owner` ）|全リソースへの完全なアクセス権。プロジェクトの課金情報を設定する。|
|編集者（ `roles/editor` ）|全リソースへの編集アクセス権。閲覧者権限と既存のリソースに対する変更の権限。|
|閲覧者（ `roles/viewer` ）|全リソースへの読み取りアクセス権。状態に影響しない読み取り専用アクションに必要な権限。|
|参照者（ `roles/browser` ）| GCP リソースを参照するためのアクセス権。プロジェクトのリソースを表示する権限はない。|

Identity and Access Management API または gcloud コマンドライン ツールを使用して、プロジェクトのメンバーにオーナーロールを付与することはできない。オーナーは、 **Cloud Console を使用することによってのみ** プロジェクトに追加できます。

事前定義ロールおよびカスタムロールのメタデータを確認する場合、 `gcloud iam roles describe ROLE_ID` コマンドで確認する。  
[事前定義ロールの一覧](https://cloud.google.com/iam/docs/understanding-roles?hl=ja#predefined_roles)

### 3.4. リソース階層/「何に対して」

[リソース](https://cloud.google.com/billing/docs/concepts#resource_overview) は「 **組織 > フォルダ > プロジェクト > リソース** 」のような階層構造に **まとめる** ことができる。

- 組織
    - Google Cloud リソース階層のルートノード
    - 組織に適用されるポリシーは、組織のすべてのリソースの階層全体に適用される
    - 組織管理者はすべてのリソースを集中てきに管理する
- フォルダ
    - プロジェクトをグループ化し、まとめて管理
    - フォルダ内にはプロジェクトや他のフォルダを作成できる
    - 組織構造に合わせて使用できる
- プロジェクト
    - ポリシーは基本的にプロジェクト以上の階層に割り当てる
    - そして、上位に設定されたポリシーは下位に継承される
- リソース
    - （省略）

例えば、「アカウント A に compute.instances.get 権限を 組織 X に対して付与」すると、「アカウント A は 組織 X に含まれる全てのフォルダの全てのプロジェクトの VM インスタンス情報を取得できる」ようになる。

### 3.5. アクセス制御のベストプラクティス

[ベストプラクティス](https://cloud.google.com/iam/docs/resource-hierarchy-access-control?hl=ja#best_practices)

- **最小権限の原則** （より細かいロールで必要最低限の権限のみ付与する）に従って Cloud IAM のロールを付与する
- **事前定義ロール** を使用する（適切な事前定義ロールがない場合のみ **カスタムロール** を使用する）
- リソースレベルではなく、 **組織またはプロジェクト** レベルでポリシーを設定する
- 個々のユーザではなく、 **グループ** にロールを付与する
    - `gcloud iam roles copy` コマンドを利用して既存プロジェクトのロールをグループにコピーして使ったりする
- ラベルを使って、リソースのアノテーションをつけ、グループ化・フィルタリングを行う
- 異なる権限を必要とする複数サービスがある場合は、サービス毎に個別のサービスアカウントを作成し、必要な権限のみ付与する

### 3.6. [リソースへのアクセス権の付与、変更、取り消し](https://cloud.google.com/iam/docs/granting-changing-revoking-access?hl=ja)

`gcloud` コマンドによるメンバのリソースに対するロールの付与について記載する。  
以降のパラメータについては下記の通り。

- GROUP： `projects` または `organizations` 。
- RESOURCE：対象リソースの名前。
- MEMBER：ロールを付与する対象となるメンバの識別子。 `member-type:id` の形式。（例： `user:my-user@example.com` ）
- ROLE_ID：ロールの名前。（ [基本ロール](https://cloud.google.com/iam/docs/understanding-roles?hl=ja#basic) 、 [事前定義ロール](https://cloud.google.com/iam/docs/understanding-roles?hl=ja#predefined_roles) ）
- FORMAT：json か yaml 。
- FILE_PATH：ファイルのパス。
- PROJECT_ID：プロジェクト ID 。

#### 3.6.1. メンバにロールを付与する

```bash
$ gcloud GROUP add-iam-policy-binding RESOURCE --member=MEMBER --role=ROLE_ID

# 例：プロジェクト my-project のユーザー my-user@example.com に閲覧者の役割を付与する
$ gcloud projects add-iam-policy-binding my-project --member=user:my-user@example.com --role=roles/viewer
```

#### 3.6.2. メンバからロールを削除する

```bash
$ gcloud GROUP remove-iam-policy-binding RESOURCE --member=MEMBER --role=ROLE_ID

# 例：プロジェクト my-project のユーザー my-user@example.com から閲覧者のロールを取り消す
$ gcloud projects remove-iam-policy-binding my-project --member=user:my-user@example.com --role=roles/viewer
```

#### 3.6.3. ポリシーの取得

```bash
$ gcloud projects get-iam-policy PROJECT_ID --format=FORMAT > FILE_PATH

# 例
$ gcloud projects get-iam-policy my-project --format json > ~/policy.json
{
  "bindings": [
    {
      "role": "roles/owner",
      "members": [
        "user:fatima@example.com"
      ]
    },
    {
      "role": "roles/editor",
      "members": [
        "serviceAccount:service-account-13@appspot.gserviceaccount.com",
        "user:wei@example.com"
      ]
    }
  ],
  "etag": "BwUjMhCsNvY=",
  "version": 1
}
```

例の通り、ロールとそのロールが付与されたメンバの配列で表現される。

#### 3.6.4. ポリシーの設定

`gcloud projects get-iam-policy` で取得したポリシーの JSON/YAML を編集するなどして、以下のコマンドでポリシーを更新することができる。

```bash
$ gcloud projects set-iam-policy PROJECT_ID FILE_PATH

# ファイル例
{
  "bindings": [
    {
      "role": "roles/owner",
      "members": [
        "user:fatima@example.com"
      ]
    },
    {
      "role": "roles/editor",
      "members": [
        "serviceAccount:service-account-13@appspot.gserviceaccount.com",
        "user:wei@example.com"
      ]
    }
  ]
}
```

## 4. リソースの管理

ここまでで出たリソースを管理してみる。  
基本 `gcloud` コマンドでの管理とする。

### 4.1. プロジェクト

```bash
$ gcloud projects COMMAND [GCLOUD_WIDE_FLAG ...]

$ gcloud projects --help # コマンドの help
```

`gcloud projects` コマンドには「プロジェクトに対する CRUD」と「プロジェクトの IAMに関する操作」を実施できる。  
[Qwiklabs と Google Cloud の概要](https://www.qwiklabs.com/focuses/2794?parent=catalog) のラボ環境 Cloud Shell などを利用して実際に実行してみる。

```bash
# プロジェクトの作成
$ gcloud projects create [PROJECT_ID]

# プロジェクトの一覧を取得 # プロジェクトが作成されていることを確認する
$ gcloud projects list
PROJECT_ID                    NAME                          PROJECT_NUMBER
qwiklabs-gcp-02-eac86d9938ef  qwiklabs-gcp-02-eac86d9938ef  264608401336 # これが作成したプロジェクトとする

# プロジェクトの IAM ポリシーを取得 # 権限毎にユーザ・グループ・サービスアカウントが表示される
$ gcloud projects get-iam-policy qwiklabs-gcp-02-eac86d9938ef
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-02-eac86d9938ef@qwiklabs-gcp-02-eac86d9938ef.iam.gserviceaccount.com
  - user:student-02-843d65d5d79e@qwiklabs.net # 現在のユーザ
  role: roles/editor
- members:
  - serviceAccount:qwiklabs-gcp-02-eac86d9938ef@qwiklabs-gcp-02-eac86d9938ef.iam.gserviceaccount.com
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-02-eac86d9938ef@qwiklabs-gcp-02-eac86d9938ef.iam.gserviceaccount.com
  - user:student-02-843d65d5d79e@qwiklabs.net # 現在のユーザ
  role: roles/viewer
etag: BwW_ToiGehg=
version: 1

# メンバーにプロジェクトの権限を付与する # なお、リソースはプロジェクトなのでロールは基本ロールの「'roles/browser'」「'roles/editor'」「'roles/owner'」のみ
gcloud projects add-iam-policy-binding [PROJECT_ID] --member=MEMBER --role=ROLE
```

## 5. 監視

[Google Cloud のオペレーション スイート](https://cloud.google.com/products/operations?hl=ja)

- Cloud Logging
    - リアルタイムでログを管理して分析する、大規模なフルマネージドサービス
    - アプリケーションとシステムのログデータのほか、GKE 環境、VM、Google Cloud サービスからのカスタム ログデータも取り込める
    - 選択したログを分析でき、アプリケーションのトラブルシューティングが加速する
- Cloud Monitoring
    - 組み込みの大規模な指標オブザーバビリティ
    - クラウドで実行されるアプリケーションのパフォーマンスや稼働時間、全体的な動作状況を確認できる
    - メトリクス・イベント・メタデータを収集し、チャートやダッシュボードで可視化してアラート管理できる

### 5.1. Cloud Logging

Cloud Logging は IAM を使用して Google Cloud リソースのロギングデータへの [アクセス制御](https://cloud.google.com/logging/docs/access-control?hl=ja#permissions_and_roles) を実施できる。

- `roles/logging.viewer`（ログビューア）（＝ `roles/viewer` ）
    - アクセスの透明性ログとデータアクセス監査ログ以外のすべての Logging 機能に対する読み取り専用権限
- `roles/logging.privateLogViewer`（プライベート ログビューア）
    - `roles/logging.viewer` に加え、アクセスの透明性ログとデータアクセス監査ログの読み取り権限
    - このロールは、_Required バケットと _Default バケットにのみ適用される
- `roles/logging.logWriter`（ログ書き込み）
    - サービスアカウントに付与して、ログの書き込みに十分な権限のみをアプリケーションに付与できる
    - ただし、閲覧権限はない
- `roles/logging.bucketWriter`（ログバケット書き込み）
    - サービス アカウントに付与すると、ログバケットにログを書き込むための十分な権限を Cloud Logging に付与できる
    - このロールを特定のバケットに制限するには、IAM 条件を使用する
- `roles/logging.configWriter`（ログ構成書き込み）は
    - ログベースの指標、除外、バケット、ビューを作成し、シンクをエクスポートする権限
    - これらのアクションに Logs Explorer（コンソール）を使用するには、 `roles/logging.viewer` を追加する
- `roles/logging.admin`（Logging 管理者）
    - Logging に関連するすべての権限
- `roles/logging.viewAccessor`（ログ表示アクセス者）
    - ビュー内のログを読み取るための権限
    - このロールを特定のバケット内のビューに制限するには、IAM 条件を使用
- `roles/editor`（プロジェクト編集者）
    - roles/logging.viewer の権限、ログエントリの書き込み権限、ログの削除権限、ログベースの指標の作成権限が含まれる
    - この役割では、エクスポート シンクの作成や、アクセスの透明性ログまたはデータアクセス監査ログの読み取りを行うことはできない
- `roles/owner`（プロジェクト オーナー）
    - Logging に対する完全アクセス権（アクセスの透明性ログとデータアクセス監査ログ）

#### 5.1.1. 監査ログ

権限管理された上で、オペレーションを監視するログに以下の種類がある。（ [Cloud Audit Logs](https://cloud.google.com/logging/docs/audit) ）

- 管理アクティビティ監査ログ
    - リソース構成やメタデータ変更オペレーションが記録される
    - デフォルトで有効で無効にできない
- データアクセス監査ログ
    - リソース構成やメタデータ読み取り API 操作が記録される
    - デフォルトは無効
- システムイベント監査ログ
    - Google システムによって生成される管理アクション
- ポリシー拒否監査ログ
    - セキュリティ ポリシー違反のため Google Cloud サービスがユーザーまたはサービス アカウントへのアクセスを拒否したときに、ポリシー拒否監査ログを記録する

### 5.2. Cloud Monitoring

- [ワークスペース](https://cloud.google.com/monitoring/workspaces?hl=ja#accounts) は、1 つ以上の Google Cloud プロジェクトまたは AWS アカウントに含まれるリソースをモニタリングするためのツール
    - 各ワークスペースには、Google Cloud プロジェクトや AWS アカウントなど、1～100 個のモニタリング対象プロジェクトを作成できる
    - ワークスペースの数に制限はありませんが、Google CloudプロジェクトとAWSアカウントを複数のワークスペースでモニタリングすることはできない
- **ホストプロジェクト**
    - すべてのワークスペースには、ホストプロジェクトがある
    - ワークスペースの作成に使用した Google Cloud プロジェクトが、ワークスペースのホスト プロジェクト
- モニタリング対象プロジェクト
    - ホストプロジェクトでワークスペースを作成すると、Google Cloud プロジェクトと AWS アカウントが追加できるようになる
    - 新しい空白の Google Cloud プロジェクトを使用してワークスペースをホストしてから、モニタリングするプロジェクトと AWS アカウントをワークスペースに追加するのがよい
        - ホスト プロジェクトと Workspace の名前を自由に選択でき、ワークスペース間でモニタリング対象プロジェクトを柔軟に移動できる

## 6. コスト管理

### 6.1. コスト算出

[Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator/?hl=ja) で各サービスに適した算出が可能。  
ただし、 BigQuery など一部特殊なものもある。  
請求データの分析は [Cloud Billing データを BigQuery にエクスポートする](https://cloud.google.com/billing/docs/how-to/export-data-bigquery?hl=ja#differences_between_exported_data_and_invoices) を参照。

### 6.2. 請求

請求先アカウントのリンク（紐付け）を更新するためには、 **請求先アカウント管理者** および **プロジェクト支払い管理者** の権限が必要になる。

- [Cloud Billing のコンセプト](https://cloud.google.com/billing/docs/concepts?hl=ja#billing_account)
- [プロジェクトの請求設定の変更](https://cloud.google.com/billing/docs/how-to/modify-project)

## 7. Google Cloud サービス

Cloud Console から Google Cloud が提供するサービスを利用することができ、以下のカテゴリで構成される。（[詳細](https://cloud.google.com/docs/overview/cloud-platform-services#top_of_page)）

- コンピューティングとホスティング
- ストレージ
- データベース
- ネットワーキング
- ビッグデータ
- 機械学習

- [プロダクトとサービスの一覧](https://cloud.google.com/products?hl=ja)
- [AWS/Azure/GCPサービス比較 2021.04](https://qiita.com/hayao_k/items/906ac1fba9e239e08ae8)

以下、他ページに現状整理し辛いサービスを記載する。

### Cloud Deployment Manager

AWS でいう CloudFormation 。  
単一の API に対応した **リソース** とそのリソースをまとめた **デプロイメント** などで構成される。  
`gcloud deployment-manager deployments` コマンドでデプロイメントを管理し、 `gcloud deployment-manager resouces` コマンドでデプロイメントに含まれるリソースを管理する。
