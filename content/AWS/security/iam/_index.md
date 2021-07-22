---
title: "IAM"
date: 2021-01-30T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

[AWS](https://docs.aws.amazon.com/ja_jp/index.html) [Identity and Access Management（ IAM ）](https://docs.aws.amazon.com/ja_jp/iam/) についてまとめる。

<!--more-->

# 概要

AWS IAM は AWS リソースをセキュアに操作するために、認証・認可の仕組みを提供する。

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

## AWS リソースにアクセスする流れ・仕組み

1. プリンシパル
    - アクセスの主体：ユーザー/グループ、ロール、アプリケーション
2. 認証
    - プリンシパルの認証
3. リクエスト
    - アクション、リソース、プリンシパル、環境データ、リソースデータ
4. 認可
    - ポリシーベースの許可
    - ID ベースポリシー、リソースベースポリシー、その他ポリシー
5. アクション/オペレーション
6. AWS リソース

## 用語

-  ユーザー
    - **IAM ユーザー**
        - IAMユーザーはAWSアカウントの配下に作成され、**AWSマネジメントコンソール**にアクセスするための独自のID/Passwordを割り当てることができる
        - IAMユーザーの単位で**アクセスキー**、**シークレットアクセスキー**が払い出される
    - **フェデレーションユーザー**
        - フェデレーションユーザーは、IAMユーザーとは異なり、AWSアカウントに永続的なIDを持っていない
        - フェデレーションユーザーは外部の**IDプロバイダ**（**IdP**）が提供するID/Passwordを指す
        - IDプロバイダにはOpenID ConnectやSAML 2.0等が挙げられる
        - フェデレーションユーザーに**ロール**と呼ばれるエンティティをIAM上で作成・アタッチすることでアクセス・権限のコントロールを行う
- **IAM グループ** ：IAMユーザの集合
- **ロール** ：AWSリソース、フェデレーションユーザーに割り当てる役割
    - あるAWSリソースが自分自身・別のAWSリソースにアクセスするためにはアクセス権限が必要となる
    - 例えば、あるEC2上のアプリケーションが別のAWSリソースにアクセスする場合、そのAWSリソースにアクセスする権限をもったロールである必要がある
- **ポリシー** ：IAMユーザー・IAMグループ・ロールに割り当てるアクセス許可
    - AWS管理ポリシー
        - AWSがあらかじめ作成し、IAM上で提供してくれているポリシー
    - カスタマー管理ポリシー
        - ユーザーが作成・管理するポリシー
        - AWS管理ポリシーには存在せず、且つ、好きなポリシーを作ることができる
    - インラインポリシー
        - IAMユーザー・IAMグループ・ロール内に作成するポリシー
        - カスタマー管理ポリシーとは異なり、他のIAMユーザー・IAMグループ・ロールへ再利用することはできない
- **IDプロバイダ**（**IdP**）
    - OpenID Connect、SAML等のAWS外に存在するID/Passwordによる認証サービス提供者
    - IAM上の「IDプロバイダ」にて外部のIDプロバイダと連携設定が可能

### Amazon Resource Name （ ARN ）

**ARN** は Amazon Resource Name の略で AWS リソースを一意に識別する値。

```
arn:partition:service:region:account-id:resource
arn:partition:service:region:account-id:resourcetype/resource
arn:partition:service:region:account-id:resourcetype:resource
```

- `partition`
    - リソースが配置されるパーティション
    - 標準のAWSリージョンの場合、パーティションは **aws**
- `service`
    - AWSのサービス名
    - 「apigateway」や「ec2」など（ [参考](https://docs.aws.amazon.com/ja_jp/general/latest/gr/aws-arns-and-namespaces.html#genref-aws-service-namespaces) ）
- `region`
    - リソースが配置されるリージョン
    - 東京リージョンであれば「ap-northeast-1」（ [参考](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) ）
- `account-id`
    - リソースを保有している AWS アカウントID（数字のみ）
    - 一部のARNはこのアカウント番号を必要としないため省略される
- `resource` `resourcetype/resource` `resourcetype/resource`
    - サービスにより様々
    - リソースタイプの指標（IAMユーザやRDSなど）が含まれることがよくある
    - さらにスラッシュ（/）またはコロン（:）、リソース名自体が続く
    - 一部のサービスではパス（ファイルパス、URIなど）を指定できる（ARNのパス）
    - [参考](https://docs.aws.amazon.com/ja_jp/general/latest/gr/aws-arns-and-namespaces.html#arns-paths)

[参考](https://docs.aws.amazon.com/ja_jp/general/latest/gr/aws-arns-and-namespaces.html)

> ### AWS Directory Serviceとの連携
> IAMは**AWS Directory Service**と連携してロールを作成することによって、**Simple AD**などで管理するID/Passwordを利用することができる。（[詳細](https://docs.aws.amazon.com/ja_jp/directoryservice/latest/admin-guide/ms_ad_manage_roles.html)）

# ID と認証情報の管理

## AWS アカウントのルートユーザーアクセスキーをロックする

- AWS アカウントのルートユーザー
    - 完全なアクセス権を持つユーザー
    - AWS アカウント作成時のメールアドレス/パスワードでサインイン
    - IAM で設定するアクセスポリシーでアクセス許可を制限できない（ AWS Organizations の SCP で可能）
    - 基本的に使用しない、 IAM ユーザーを作成する
- アクセスキー
    - ルートユーザー・ IAM ユーザーに与える認証情報
    - アクセスキー ID 、シークレットアクセスキーで構成される
    - AWS CLI や AWS SDK 等からのアクセスに署名
    - 最大 2 つ持てる
    - プラクティス：ルートユーザーのアクセスキーは削除する！

## 個々の IAM ユーザーを作成

- IAM ユーザー
    - AWS で作成するユーザーまたはアプリケーション
    - 名前と認証情報で構成される
    - IAM ユーザーを識別する方法
        - プレンドリ名：人がわかりやすい名前
        - ARN（ Amazon Resource Name ）：リソースポリシーの Principal 要素で指定
        - ユーザーの一意の識別子：フレンドリ名を再利用したいとき等に権限のエスカレーションを避けることができる
            - 普段見ない、AWS CLI とかから意識的に取得しないと見れない
    - 認証情報
        - コンソールパスワード、アクセスキー

## ユーザーの強力なパスワードポリシーを設定

- パスワードに要求される強度とパスワード管理のポリシーを設定可能
    - パスワードに利用する文字に関するルール
    - ユーザー自身によるパスワード変更の許可
    - 有効期限、再利用禁止の世代数など
    - ルートユーザーには適用されない

## アクセスキーを共有しない

- 個々人ごとに IAM ユーザーを作成する
- 個々のアプリケーションごとに IAM ロールを作成する
- アクセスキーの置き場所に注意

## 特権ユーザーに対して MFA を有効化する

- MFA：他要素認証
    - パスワード、アクセスキーに加えてセキュリティを強化する仕組み

# アクセス権限の管理

## AWS 管理ポリシーを使用したアクセス許可の使用

- ポリシー
    - エンティティ（IAM アイデンティティや AWS リソース）に関連つけることによってアクセス許可を定義できるオブジェクト
    - JSON ポリシードキュメントで条件を記述
    - 1 つ以上の Statement ブロックで構成
- ポリシータイプ
    - アイデンティティベース
        - 管理ポリシー： AWS 管理ポリシー、カスタマー管理ポリシー（自作のポリシー）
            - ポリシー自体を管理し、 エンティティに関連づけて利用する
        - インラインポリシー
            - エンティティに埋め込んで利用し、再利用不可能
            - できるだけ使用せず、カスタマー管理ポリシーを使用する
    - リソースベース
        - IAM ロールの信頼ポリシー
    - パーミッションバウンダリー
        - IAM アクエス許可の境界
    - アクセスコントロール（ ACL ）
        - 別アカウントのポリシーを制御
        - 唯一 JSON で記載しない
    - セッションポリシー
        - 一時的なアクセス権限を渡すために作成する（フェデレーションユーザーなど）

## 追加セキュリティに対するポリシー条件を使用

- ポリシードキュメント（ JSON ）の主な要素
    - Version
        - ポリシー言語のバージョン、最新を記載する（ `2012-10-17` ）
    - Statemant
        - アクセス許可に関する複数要素（ Effect/Action/Resource など ）を含むブロック
        - 複数作成可能
        ```
        {
            "Version": "2012-10-17",
            "Statement": {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::example_bucket"
            }
        }
        ```
    - Effect
        - アクセスを要求した結果の許可/拒否の指定 `Allow` `Deny`
        - 暗黙的な Deny（デフォルト） ＜ 明示的な Allow ＜ 明示的な Deny
    - Principal（誰が）
        - アクセスが許可/拒否される IAM エンティティを ARN 形式で記述
    - Action（何を）
        - アクセス許可/拒否されるアクション `Action` `NotAction` （[一覧](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/APIReference/API_Operations.html)）
    - Resource（何に対して）
        - アクセスが許可/拒否対象のリソースを ARN 形式で記述
    - Condition（いつ）
        - ポリシーが有効になるタイミング
        - 条件演算子・条件キーを利用して記載
- ツール
    - AWS Policy Generator
    - IAM Policy Validator
    - IAM Policy Simulator 、など
- 最小権限を付与すること

## IAM ユーザーへのアクセス許可を割り当てるためにグループを使用

- IAM グループ
    - IAM ユーザーの集合
    - IAM ユーザーは複数の IAM グループへ所属可能（最大 10）
    - IAM ユーザー個別管理が大変なので、こちらを利用する（所属グループを変更するだけ）

# 権限の委任

- IAM ロール
    - AWS サービスやアプリケーションなどに権限を付与する仕組み
    - 一時的なセキュティ認証情報：AWS Security Token Service（ STS ）
        - 有効期限付きのアクセスキーID/シークレットアクセスキー/セキュリティトークンで構成

## EC2 で実行するアプリケーションに対してロールを使用する

- アプリケーションが AWS サービスにアクセスするためには認証が必要
- EC2（OS）側に持たせる必要がない

## ロールを使用したアクセス許可の委任

- 別の AWS アカウントのユーザーが認証情報を共有せずに、自分の AWS アカウントのリソースに隠せす制御可能
- Amazon Cognito を用いたモバイルアプリの Web ID フェデレーションなどにも利用可能

# ID と権限のライフサイクル管理

## AWS アカウントのアクティビティ監視

- AWS CloudTrail
    - AWS API コール、関連イベントをロギング
    - 適切なユーザーが与えられた権限で環境を操作しているのか確認と記録に使用
- Amazon CloudWatch
- AWS Config
- アクセスアドバイザー

## 不要な認証情報を削除する

- IAM 認証情報レポート（ Credential Report ）を確認して、利用がないものは確認して削除

## 認証情報を定期的にローテーションする

- パスワードに有効期限を設定し、ユーザーに更新させる
- アクセスキーもローテーションできるようにする
    - アクセスキーは 2 個作れるので、これを利用する

# 参考

- [トラブルシューティング](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/troubleshoot.html)

# Landing Zone

Landing Zone は、Well-Architected Framework を始めとした「AWS のベストプラクティス」に基づいて構成したアカウントを、スケーラブルに展開していくための仕組みの総称。  
これにより、ガバナンスを効かせた形でアカウントを自動展開することができる。  

Landing Zoneは、下記機能から構成されます。

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

ここでポイントとなるのは、Landing Zone はマルチアカウント戦略を実現する仕組みの総称であり、AWS サービスの名称では無い。

## Landing Zone を適用する 2 つの方法

- 実装 1: AWS Control Tower
- 実装 2 : 独自実装の Landing Zone

### 実装 1: AWS Control Tower

AWS Control Tower では、ID、フェデレーティッドアクセス、マルチアカウント構成のベストプラクティスを使用して、Landing Zone をセットアップする。

- AWS Organizations を使用してマルチアカウント環境を作成する
- AWS Single Sign-On (SSO) を使用して ID 管理を提供する
- AWS SSO を使用してアカウントにフェデレーティッドアクセスを提供する
- AWS CloudTrail のログや、Amazon S3 に保存される AWS Config のログを集中管理する
- AWS IAM および AWS SSO を使用してクロスアカウントセキュリティ監査を有効化する

### 実装 2 : 独自実装の Landing Zone

もうひとつの手段として、Landing Zone の各機能を独自実装する。

### 手順の大きな流れ

体感する手順はこんな感じ。AWSアカウントも、Organizationsの機能を使って作成する。

1. rootユーザーでアクセス可能なAWSアカウントを用意する
2. Organizationsを作成する
3. OrganizationsからAWSアカウントを追加する
4. 組織ユニット（OU）を作成する
5. サービスコントロールポリシー（SCP）を作成する
6. 組織ユニットにサービスコントロールポリシーを適用する
7. AWSアカウントにアクセスし動作確認する