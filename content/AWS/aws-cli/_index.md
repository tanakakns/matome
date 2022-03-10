---
title: "AWS CLI"
date: 2021-01-30T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 10
---

1. [インストール](#1-インストール)
2. [プロファイル](#2-プロファイル)
3. [MFA認証](#3-MFA認証)

<!--more-->

## 1. インストール

```bash
$ brew install awscli
```

## 2. プロファイル

AWS CLI を実行する際、以下のような環境変数でクレデンシャルを設定することにより認証され AWS にアクセスすることができる。

```bash
$ export AWS_ACCESS_KEY_ID=<アクセスキー>
$ export AWS_SECRET_ACCESS_KEY=<シークレットキー>
$ export AWS_DEFAULT_REGION=ap-northeast-1 # 任意のリージョン
```

しかし、複数の AWS 環境があるなど、いちいち環境変数を変えるのは面倒だ。  
そういう時に **プロファイル** を利用する。

```bash
# プロファイルの作成
$ aws configure --profile <プロファイル名>
```

プロファイルを作成すると `~/.aws/credentials` と `~/.aws/config` が作成され、以下のようになる。

```bash
# ~/.aws/credentials

[default]
aws_access_key_id = <デフォルトのアクセスキー>
aws_secret_access_key = <デフォルトのシークレットキー>

[<プロファイル名>]
aws_access_key_id = <プロファイル名のアクセスキー>
aws_secret_access_key = <プロファイル名のシークレットキー>
```

```bash
# ~/.aws/config

[default]
region = <デフォルトのリージョン>
output = <デフォルトの出力形式>

[profile <プロファイル名>]
region = <プロファイル名のリージョン>
output = <プロファイル名の出力形式>
```

なお、 `--profile` オプションで `<プロファイル名>` を指定しない場合は `default` が設定される。  
各コマンドでプロファイルのクレデンシャルを指定して実行したい場合も以下のように `--profile` オプションを利用する。

```bash
$ aws s3 ls --profile <プロファイル名>
```

なお、いちいち `--profile` オプションを指定するのは面倒だ。  
そんな時は、以下の環境変数を指定すると `--profile` オプションを省略してもプロファイルが適用される。

```bash
$ export AWS_DEFAULT_PROFILE=<プロファイル名>
# もしくは
$ export AWS_PROFILE=<プロファイル名>
```

## 3. MFA認証

AWS CLI で MFA 認証が必要な場合、通常、以下を実施する必要がある。

- [AWS CLI 経由で MFA を使用してアクセスを認証する](https://aws.amazon.com/jp/premiumsupport/knowledge-center/authenticate-mfa-cli/)

しかし、超面倒だ。  
そんな時は [aws-mfa](https://github.com/broamski/aws-mfa) というサポートツールを利用することで少し楽になる。

### 3.1. aws-mfa を利用した MFA 認証

インストールは以下。

```bash
$ pip install aws-mfa
or
$ pip3 install aws-mfa
```

`~/.aws/credentials` ファイルに以下を追記する。

```bash
[<プロファイル名>-long-term]
aws_access_key_id = <あなたのIAM Userのアクセスキー>
aws_secret_access_key = <あなたのIAM Userのシークレットキー>
aws_mfa_device = arn:aws:iam::312574223261:mfa/<あなたのIAM User ID>
```

プロファイル名に `-long-term` がついているのがポイント。  
上記が出来たら、以下のコマンドを実行する。

```bash
$ aws-mfa --profile <プロファイル名>
```

ワンタイムトークンの入力が求められ、入力後 `~/.aws/credentials` に以下が追記される。

```bash
[<プロファイル名>]
assumed_role = False
aws_access_key_id = <一時的なアクセスキー>
aws_secret_access_key = <一時的なシークレットキー>
aws_session_token = <一時的なセッショントークン>
aws_security_token = <一時的なセキュリティトークン>
expiration = <キー・トークンの有効期限>
```

`<プロファイル名>-long-term` プロファイルを作成して `aws-mfa --profile <プロファイル名>` コマンドを実行したことにより、 `<プロファイル名>` のプロファイルが一時トークン・セッションを含んだ形で作成されることになる。  
生成されたプロファイルを有効にするために以下を実行する。

```bash
$ export AWS_DEFAULT_PROFILE=<プロファイル名>
# もしくは
$ export AWS_PROFILE=<プロファイル名>
```

以上でトークン・セッションの有効期限内においては AWS CLI が利用可能になる。  
有効期限が切れたら、 `aws-mfa --profile <プロファイル名>` を再実行すればよい。