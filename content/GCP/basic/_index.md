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

1. [Qwiklabs](#1-Qwiklabs)
2. [Cloud Console](#2-cloud-console)

<!--more-->

## 1. Qwiklabs

[Qwiklabs](https://www.qwiklabs.com/?locale=ja)は、 GCP と AWS のラボ環境を一時的に払い出してくれて、ハンズオンができるサービス。

### 1.1. ラボ

トピックと専門知識のレベルにかかわらず、Qwiklabs のラボのインターフェースはどれも同じで、以下のような機能・コンポーネントがある。

- ラボを開始（ボタン）
  - このボタンをクリックすると、実践演習に必要なすべてのサービスと認証情報が有効になった Google Cloud 環境が一時的に作成される
  - この環境を使用して実践演習を行う
  - また、このボタンを押すと、カウントダウンタイマーも起動する
- クレジット
  - ラボの料金で、1 クレジットは通常 1 米ドルに相当
  - 入門レベルのラボには無料で受講できるものもある
  - 専門性が高いラボほど料金が高くなり、これは、タスクの処理内容が高度になり、必要な Google Cloud リソースが増えるため
- 時間
  - ラボは実行できる時間決まっており、 「ラボを開始」ボタンをクリックするとタイマーのカウントダウンが始まり、00:00:00 になると、一時的に作成された Google Cloud 環境とリソースが削除される
- スコア
  - 多くのラボにスコアが備わっている
  - これは「アクティビティ トラッキング」と呼ばれる機能で、ラボに含まれる所定の手順を確実に完了するためのもの
  - アクティビティ トラッキング機能が付いているラボに合格するには、すべての手順を順番に完了する必要があり、そのようにしてラボを完了した場合のみ、クレジットを受け取ることができる

## 2. Cloud Console

[Cloud Console](https://cloud.google.com/cloud-console/)は、Google Cloud の開発ツールの中核であるウェブコンソール。  
ログインするにはユーザー名とパスワードが必要で、後述の[Cloud Identity and Access Management（Cloud IAM）](https://cloud.google.com/iam/)によりアクセス権限（ロール）管理される。

### 2.1. プロジェクト

Google Cloud のリソースはすべて、[プロジェクト](https://cloud.google.com/docs/overview/#projects) に属しており、構築する対象をまとめて管理する 1 つの単位と考えることができる。  
Cloud Console 上で選択できるプロジェクトは、グローバルで一意な **プロジェクトID** で管理される。  
プロジェクトには ID の他にも、 **プロジェクト名** 、 **プロジェクト番号** があり、Google Cloud サービスを操作するときは、これらの識別子を頻繁に使用する。

プロジェクトにはリソースとサービス（仮想マシンのプール、データベース セット、それらを接続するネットワークなど）が含まれる。さらに各種設定と権限も含まれており、これを使用して、セキュリティルールと誰がどのリソースにアクセスできるかが指定される。  
大企業の、あるいは経験豊富な Google Cloud ユーザーの場合、Google Cloud のプロジェクトが数十から数千あるのは珍しくなく、組織によって Google Cloud の使用方法は異なるため、プロジェクトを使ってチーム別やプロダクト別などでクラウドコンピューティングサービスをまとめて管理する。  
また、[共有 VPC](https://cloud.google.com/vpc/docs/shared-vpc?hl=ja) または [VPC ネットワークピアリング](https://cloud.google.com/vpc/docs/vpc-peering?hl=ja)を使用しない限り、あるプロジェクトが別のプロジェクトのリソースにアクセスすることはできない。  
さらに、課金を有効にすると、各プロジェクトは 1 つの請求先アカウントに関連付けられる。

### 2.2. Google Cloud サービス

Cloud Console から Google Cloud が提供するサービスを利用することができ、以下のカテゴリで構成される。（[詳細](https://cloud.google.com/docs/overview/cloud-platform-services#top_of_page)）

- コンピューティングとホスティング
- ストレージ
- データベース
- ネットワーキング
- ビッグデータ
- 機械学習

[プロダクトとサービスの一覧](https://cloud.google.com/products?hl=ja)

### 2.3. ロールと権限

前述したように、Google Cloud には、クラウドコンピューティングサービスの他に、誰がどのリソースにアクセスできるかを定義する権限とロールのコレクションもある。  
[Cloud Identity and Access Management（Cloud IAM）](https://cloud.google.com/iam/) サービスを使って、このようなロールと権限を管理できる。

Google Cloud には 3 つの [基本ロール](https://cloud.google.com/iam/docs/understanding-roles/#primitive%5C_roles) がある。  
基本ロールは **プロジェクトレベル** の権限を設定し、別途指定しない限り、すべての Google Cloud サービスへのアクセスと管理を制御する。

|ロール名|権限|
|:---|:---|
|roles/viewer（閲覧者）|既存のリソースやデータを表示する（ただし変更は不可能）など、状態に影響しない読み取り専用アクションに必要な権限。|
|roles/editor（編集者）|すべての閲覧者権限と、状態を変更するアクション（既存のリソースの変更など）に必要な権限。|
|roles/owner（オーナー）|すべての編集者権限と、次のアクションを実行するために必要な権限: プロジェクトおよびプロジェクト内のすべてのリソースの権限とロールを管理する、プロジェクトの課金情報を設定する。|

### 2.4. API とサービス

Google Cloud API は、ビジネス管理から機械学習にまで及ぶさまざまな分野の API が 200 以上用意されており、サービスと同様に、すべて Google Cloud プロジェクト / アプリケーションと簡単に統合できる。  
[API 設計ガイド](https://cloud.google.com/apis/design/) で説明されているリソース指向の設計原則が適用されている。  
なお、 Cloud API を利用する際は、各自で有効に設定する必要がある。（サイドメニュー「API とサービス」 -> 「ライブラリ」から APIを検索して「有効にする」）

- [Google APIs Explorer](https://developers.google.com/apis-explorer/#p/)

### 2.5. Cloud Shell

[Cloud Shell](https://cloud.google.com/shell/docs/features) はブラウザで機能するコマンド実行環境。  
その実態は Debian ベースの仮想マシンで、さまざまな開発ツール（gcloud、git など）や、永続的な 5 GB のホーム ディレクトリが用意されている。  
ターミナルにコマンドを入力して、Google Cloud プロジェクトのリソースやサービスを管理できる。  
Cloud Shell を使用すると、Cloud Console を離れずにすべてのシェルコマンドを実行できる。  
Google Cloud Console のタイトルバーで、「Cloud Shell をアクティブにする」 をクリックすることで利用できる。

```bash
$ gcloud auth list

Credentialed Accounts
ACTIVE  ACCOUNT
*       student-00-918550b6c355@qwiklabs.net
To set the active account, run:
    $ gcloud config set account `ACCOUNT`
```

Cloud Shell には固有のコマンドラインツールがあらかじめ実装されおり、メインの Google Cloud ツールキットは [gcloud](https://cloud.google.com/sdk/gcloud/) で、リソース管理やユーザー認証など、プラットフォーム上のさまざまなタスクに使用されます。  
今実行したのは `gcloud` コマンドの [auth list](https://cloud.google.com/sdk/gcloud/reference/auth/list) で、有効なアカウント名を一覧表示する。  
Cloud Shell には、あらかじめインストールされてたツールキットのほかに、Unix の標準コマンドや、nano などのテキスト エディタも備わっている。

```bash
# プロジェクト ID 一覧
$ gcloud config list project
[core]
project = qwiklabs-gcp-44776a13dea667a6
```

- [gcloud コマンドライン ツールの概要](https://cloud.google.com/sdk/gcloud)
- [gcloud リファレンス](https://cloud.google.com/sdk/gcloud/reference/?hl=ja)