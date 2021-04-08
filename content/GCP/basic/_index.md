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
5. [Google Cloud サービス](#4-google-cloud-サービス)

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

- [Google APIs Explorer](https://developers.google.com/apis-explorer/#p/)

## 3. リソース階層とアクセス制御

### 3.1. Cloud Identity and Access Management（Cloud IAM）

[Cloud Identity and Access Management（Cloud IAM）](https://cloud.google.com/iam/) は、「誰が」「どういう操作を」「何に対して」行う権限を持つかを管理するサービス。

|制御内容|具体例|
|:---|:---|
|誰が|メンバー|
|どういう操作を|ロール|
|何に対して|リソース|

Cloud Console の 「サイドメニュー」->「 IAM と管理」->「 IAM 」から利用できる。

### 3.2. メンバー

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
    - サービスアカウントにうよって使われるキーは自動的に管理される
    - ユーザがキーを作成・ダウンロードすることも可能

### 3.3. 権限とロール

リソースに対する操作の権限は以下の命名規則で表現される。

```zsh
<サービス名>.<リソース>.<動詞>

ex)
compute.instance.delete
compute.instance.start
compute.instance.setMachineType
```

### 3.4. ロールの種類

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

### 3.5. リソース階層

リソースは「 **組織 > フォルダ > プロジェクト > リソース** 」のような階層構造になっている。

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

### 3.6. アクセス制御のベストプラクティス

- **最小権限の原則** （より細かいロールで必要最低限の権限のみ付与する）に従って Cloud IAM のロールを付与する
- **事前定義ロール** を使用する（適切な事前定義ロールがない場合のみ **カスタムロール** を使用する）
- リソースレベルではなく、 **組織またはプロジェクト** レベルでポリシーを設定する
- 個々のユーザではなく、 **グループ** にロールを付与する
- ラベルを使って、リソースのアノテーションをつけ、グループ化・フィルタリングを行う
- 異なる権限を必要とする複数サービスがある場合は、サービス毎に個別のサービスアカウントを作成し、必要な権限のみ付与する

### 3.7. 監査ログ

- 管理アクティビティ監査ログ
    - リソース構成やメタデータ変更オペレーションが記録される
    - デフォルトで有効で無効にできない
- データアクセス監査ログ
    - リソース構成やメタデータ読み取り API 操作が記録される
    - デフォルトは無効
- システムイベント監査ログ
    - Google システムによって生成される管理アクション

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

## 5. Google Cloud サービス

Cloud Console から Google Cloud が提供するサービスを利用することができ、以下のカテゴリで構成される。（[詳細](https://cloud.google.com/docs/overview/cloud-platform-services#top_of_page)）

- コンピューティングとホスティング
- ストレージ
- データベース
- ネットワーキング
- ビッグデータ
- 機械学習

[プロダクトとサービスの一覧](https://cloud.google.com/products?hl=ja)