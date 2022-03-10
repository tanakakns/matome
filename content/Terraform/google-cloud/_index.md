---
title: "Google Cloud"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 2
---

Google Cloud で Terraform を利用するための初期設定方法を記載する。  

<!--more-->

Google Cloud コンソールへログインし Cloud Shell にて以下を実行する。（権限管理は Owner にしか実施できないので注意）

```bash
$ gcloud config set project <プロジェクト名>
$ gcloud iam service-accounts create terraform --display-name "Used by Terraform"
$ gcloud projects add-iam-policy-binding <プロジェクト名 \
  --member serviceAccount:terraform@<プロジェクト名>.iam.gserviceaccount.com \
  --role roles/editor
# なお、 terraform で IAM 関係の操作を行う場合は追加で以下のような権限を付与してやる必要がある。
$ gcloud projects add-iam-policy-binding <プロジェクト名> \
  --member=serviceAccount:terraform@<プロジェクト名>.iam.gserviceaccount.com \
  --role=roles/resourcemanager.projectIamAdmin
```

Google Cloud コンソールにて上記で作成したサービスアカウントから JSON 形式で秘密鍵を作成しファイルをダウンロードする。  
ダウンロードしたファイルをローカルマシンの任意の場所に配置し、 以下のように環境変数の設定を行う。

```bash
$ export GOOGLE_APPLICATION_CREDENTIALS=<path to downloaded json file>
```

Google Cloud コンソールにて「tfstate-<プロジェクト名>」のような Cloud Storage バケットを公開アクセス禁止で作成し、以下のように `backend.tf` を作成する。

```terraform
terraform {
  backend "gcs" {
    bucket = "tfstate-<プロジェクト名>"
    prefix = "terraform"
  }
}
```

あとはその他 Terraform で必要ファイルを作成して、 `terraform init` から始める。