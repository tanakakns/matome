---
title: "AWS IAM"
date: 2023-06-27T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 5
---

AWS IAM （Identity and Access Management）は、 AWS サービスで「認証」と「認可」の設定を行うことができるサービスです。

<!--more-->

# 概要

AWS IAM は AWS リソースをセキュアに操作するために、認証・認可の仕組みを提供するサービスです。

- ID と認証情報の管理
  - IAM ユーザーの作成とパスワードポリシー・MFA
  - アクセスキー管理
- アクセス権限の管理
  - IAM ユーザーに対してポリシーを使用した AWS リソースへのアクセス許可
  - グループ単位のアクセス許可
- 権限の委任
  - EC2 などの AWS リソースへのロール付与
  - ロールに対するアクセス許可
- ID と権限のライフサイクル管理
  - AWS アカウントのアクティビティ監視
  - 認証情報の定期的なローテーション含めた管理

# 用語

## プリンシパル

AWS リソースにアクセスするユーザーやアプリケーションのことです。

## **IAM ユーザー**

IAM ユーザーは AWS アカウントの配下に作成され、 **AWSマネジメントコンソール** にアクセスするための ID/Password を割り当てることができます。  
IAM ユーザーの単位で **アクセスキー** 、 **シークレットアクセスキー** などのキーを払い出すことができ、 AWS CLI 実行時などの認証・認可に利用できます。

## **フェデレーションユーザー**

フェデレーションユーザーは、 IAM ユーザーとは異なり、 AWS アカウント配下で管理される ID ではありません。  
フェデレーションユーザーは外部の　**IDプロバイダ**（**IdP**）が提供する ID/Password です。  
ID プロバイダには OpenID Connect や SAML 2.0 などが挙げられます。  
フェデレーションユーザーに **IAM ロール** を IAM 上で作成・アタッチすることでアクセス・権限のコントロールを行うことができます。

## **IAM ユーザーグループ**

IAM ユーザーの集合です。

## **IAM ロール**

IAM ロールは、 **通常は AWS リソースへアクセス権限の無い** ユーザー、アプリケーション、AWS リソース（EC2 など）に役割を **移譲** するために利用します。  
そのため、 IAM ユーザーやグループに権限を与える際は IAM ロールは利用せず、直接ポリシーをアタッチすることになります。  
IAM ユーザ・グループと同様 IAM ロールにも **ポリシー** を割り当てることができ、このポリシーでアクセスコントロールを行います。

## **ポリシー**

IAM ユーザー・IAM グループ・ IAM ロールに割り当てるアクセス許可で、 JSON 形式で管理されます。  
ポリシーには以下の種類があります。

- AWS 管理ポリシー
  - AWSがあらかじめ作成し、IAM 上で提供してくれているポリシー
- カスタマー管理ポリシー
  - ユーザーが作成・管理するポリシー
  - AWS管理ポリシーには存在せず、且つ、好きなポリシーを作ることができます
- インラインポリシー
  - IAM ユーザー・IAM グループ・IAM ロール内に作成するポリシー
  - カスタマー管理ポリシーとは異なり、他のIAMユーザー・IAMグループ・ロールへ再利用することはできません

## Amazon Resource Name （ ARN ）

**ARN** は Amazon Resource Name の略で AWS リソースを一意に識別する値です。  
以下のような形式で表現されます。

```
arn:partition:service:region:account-id:resource
arn:partition:service:region:account-id:resourcetype/resource
arn:partition:service:region:account-id:resourcetype:resource
```

- `partition`
  - リソースが配置されるパーティション（標準のAWSリージョンの場合、単に「 **aws** 」）
- `service`
  - AWSのサービス名（「apigateway」や「ec2」など）
- `region`
  - リソースが配置されるリージョン（東京リージョンであれば「ap-northeast-1」）
- `account-id`
  - リソースを保有している AWS アカウントID（数字のみ）
  - 一部の ARN はこのアカウント番号を必要としないため省略されます
- `resource` `resourcetype/resource` `resourcetype/resource`
  - サービスにより様々
  - リソースタイプの指標（IAMユーザやRDSなど）が含まれることがよくある
  - さらにスラッシュ（/）またはコロン（:）、リソース名自体が続く
  - 一部のサービスではパス（ファイルパス、URIなど）を指定できる（ARNのパス）

[参考](https://docs.aws.amazon.com/ja_jp/general/latest/gr/aws-arns-and-namespaces.html)

# IAM ロール

IAM ロールは自分の AWS アカウントの権限を移譲するためのものです。  
そして、 IAM ロールを移譲する際には、 **移譲対象を信頼** する必要があります。  
この信頼する対象に関するポリシーを **信頼ポリシー** と呼びます。  
信頼ポリシーは以下のような **AssumeRolePolicyDocument** で表現します。

```javascript
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": { // 信頼する対象
        "AWS": [ "123456789012" ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

上記は「 AWS アカウント `123456789012` を信頼する」という意味になります。  
AWS STS (AWS Security Token Service) の 「sts:AssumeRole」という機能を通じて、信頼する対象に対してロールを付与します。  
信頼ポリシーは、ロールが信頼する相手に関する情報を記載しているだけなので、何かしらの権限を付与するためにはロールに追加でポリシーをアタッチする必要があります。  
さらに例を挙げると、以下は「AWS サービス EC2 を信頼する」という意味になります。

```javascript
{
  "Version" : "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { // 信頼する対象
        "Service": [ "ec2.amazonaws.com" ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

# アクセス権限の管理

## ポリシードキュメント（ JSON ）

ポリシーは JSON ドキュメントで表現します。  
項目には以下があります。

- Version
  - ポリシー言語のバージョン（ `2012-10-17` ※最新版を記載するようにしましょう）
- Statemant
  - アクセス許可に関する要素（ Effect/Action/Resource など ）を複数含むブロック
  - 複数作成可能
- Effect
  - アクセスを要求した結果の許可/拒否の指定 `Allow` `Deny`
  - 「暗黙的な Deny（デフォルト） ＜ 明示的な Allow ＜ 明示的な Deny」の基準で評価されます
- Principal（誰が）
  - アクセスを許可/拒否される主体を ARN 形式で記述します
- Action（何を）
  - アクセスを許可/拒否されるアクション `Action` `NotAction` （[一覧](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/APIReference/API_Operations.html)）
- Resource（何に対して）
  - アクセス許可/拒否対象のリソースを ARN 形式で記述します
- Condition（いつ）
  - ポリシーが有効になるタイミング
  - 条件演算子・条件キーを利用して記載します

具体例としては以下のようになります。  
「example_bucket」という名前の S3 バケットへのアクセス許可を表すポリシーになります。

```javascript
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": "s3:ListBucket",
        "Resource": "arn:aws:s3:::example_bucket"
    }
}
```

## ポリシーの種類

ポリシーには以下の種類があります。

- AWS 管理ポリシー
  - AWS があらかじめ用意してくれているポリシー
  - 例） `arn:aws:iam::aws:policy/IAMReadOnlyAccess`
- カスタマー管理ポリシー
  - ユーザが独自に作成して命名するポリシーで、再利用可能
- インラインポリシー
  - ユーザが独自に IAM ユーザー・グループ・ロールに対して埋め込むポリシー
  - ポリシー名が無く、再利用不可能

通常は AWS 管理ポリシーを利用し、適当なものが無い場合は カスタマー管理ポリシーを利用します。  
インラインポリシーはあまりお勧めできません。

## IAM ユーザ・グループへの権限付与

IAM ユーザ・グループへポリシーをアタッチすることによって権限管理を行います。  
IAM ユーザ一人一人に対して権限管理を行うのは非効率なので、 Administoator 、 SystemOperator といった IAM グループを作成して権限を付与し、 IAM グループへ IAM ユーザを所属させることによって権限管理することをお勧めします。  
IAM ユーザーは複数の IAM グループ（最大 10 ）へ所属することが可能です。

## IAM ロールへの権限付与

他 AWS アカウントや自 AWS アカウント内の AWS サービス（例えば EC2）など、対象毎に IAM ロールを作成します。  
その IAM ロールに対して信頼ポリシー（誰を信頼するのか？）と権限を与えるポリシー（何が出来るのか？）を付与します。  
（厳密には Permissions boundary といったポリシーもありますが、ここでは割愛します。）  
以上のように作成した IAM ロールを対象に付与します。

# AWS Identity and Access Management Access Analyzer

- IAM Access Analyzer の機能は、外部エンティティと共有されている組織とアカウントのリソースを識別する
- IAM Access Analyzer は、ポリシーの文法およびベストプラクティスに対して IAM ポリシーを検証する
- IAM Access Analyzer は、AWS CloudTrail ログでのアクセスアクティビティに基づいた IAM ポリシーを生成する

# Landing Zone

Landing Zone は、Well-Architected Framework を始めとした「AWS のベストプラクティス」に基づいて構成したアカウントをスケーラブルに展開していくための仕組みの総称です。  
これにより、ガバナンスを効かせた形でアカウントを自動展開することができます。  

Landing Zone は、下記機能から構成されます。

1. アカウントの発行
    - 必要な初期設定の済んだアカウントを作成
2. 管理用権限の発行
    - 対象アカウントを管理するための権限を作成
3. 共有サービスへのアクセス (ユーザー環境に合わせて個別に実装する)
    - AD やファイルサーバー等の共有サービスや運用拠点への接続経路の確保
4. AWS ログの集約
    - 監査用ログをセキュアに一元保存
5. ガードレールの設置　
    - 実施してはいけない操作の禁止 (必須のガードレール)
    - 危険な設定の監視 (強く推奨されるガードレール、推奨のガードレール)

Landing Zone はマルチアカウント戦略を実現する仕組みの総称であり、AWS サービスの名称では無いということに注意してください。

## Landing Zone を適用する 2 つの方法

- 実装 1 : AWS Control Tower
- 実装 2 : 独自実装の Landing Zone

### 実装 1 : AWS Control Tower

AWS Control Tower では、ID、フェデレーティッドアクセス、マルチアカウント構成のベストプラクティスを使用して、Landing Zone をセットアップします。

- AWS Organizations を使用してマルチアカウント環境を作成する
- AWS Single Sign-On (SSO) を使用して ID 管理を提供する
- AWS SSO を使用してアカウントにフェデレーティッドアクセスを提供する
- AWS CloudTrail のログや、Amazon S3 に保存される AWS Config のログを集中管理する
- AWS IAM および AWS SSO を使用してクロスアカウントセキュリティ監査を有効化する

### 実装 2 : 独自実装の Landing Zone

もうひとつの手段として、Landing Zone の各機能を独自実装します。

### 手順の大きな流れ

流れは以下です。  
AWSアカウントも、Organizationsの機能を使って作成します。

1. rootユーザーでアクセス可能なAWSアカウントを用意する
2. Organizationsを作成する
3. OrganizationsからAWSアカウントを追加する
4. 組織ユニット（OU）を作成する
5. サービスコントロールポリシー（SCP）を作成する
6. 組織ユニットにサービスコントロールポリシーを適用する
7. AWSアカウントにアクセスし動作確認する
