---
title: "入門"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 1
---

超今更ながら [Terraform](https://www.terraform.io/) に入門する。  
ここでは AWS を前提とする。

<!--more-->

# 用語

|用語|意味|
|:---|:---|
|Configuration|インフラ設定のコード。要するに `*.tf` ファイルに DSL で書く Terraform のコードのこと。|
|HCL|HashiCorp Configuration Languageの略。`*.tf` ファイルで使われている DSL のこと。|
|Resource|Terraform で管理する対象の基本単位。|
|Data Source|Terraform 管理外だけど、 Terraform 内で参照したい参照専用の外部データ。|
|Provider|Resource や Data Source などを作成/更新/削除するプラグイン。aws/google/azurermなど。|
|Provisioner|リソースの作成/削除時に実行するスクリプトなどのプラグイン。local-exec/remote-exec/chefなど|
|State|Terraform が認識しているリソースの状態。 `*.tfstate` ファイルのこと。|
|Backend|State の保存先。local/s3/gcsなど。|
|Module|Resource や Data Source などを再利用可能なようにまとめた Configuration の単位。|

# 環境設定

コマンドのバージョン管理ツール [asdf](https://asdf-vm.com/#/) で導入する。

```bash
$ asdf plugin add terraform
$ asdf plugin list
terraform
$ asdf list-all terraform
$ asdf install terraform 0.14.6
$ asdf global terraform 0.14.6
$ asdf current terraform
terraform       0.14.6          /Users/tanakakns/.tool-versions
$ terraform -v
Terraform v0.14.6
```

# 基本動作

1. `.tf` ファイルを書く
2. `terraform init` で初期化
    - `.terraform` ディレクトリが作成され provider のバイナリダウンロードなどの設定が行われる
3. `terraform plan` で設定に問題無いか確認する
4. `terraform apply` で実際に適用する

# `.tf` ファイルを書く

## 基本設定

変数を管理するファイルを作成する。( `variables.tf` )

```terraform:variables.tf
variable "access_key" {}
variable "secret_key" {}
variable "region" {}
```

変数に値を設定するファイルを作成する。（ `terraform.tfvars` ）

```terraform
access_key = "access_key"
secret_key = "secret_key"
region = "ap-northeast-1"
```

上記はファイルではなく環境変数（ `TF_VAR_access_key` , `TF_VAR_secret_key` ）や `terraform` コマンドオプション（ `-var` ）からの読み込みも可能。  
AWS を Provider 設定するファイルを作成する。（ `aws.tf` ）

```terraform
terraform {
  required_version = ">= 0.12.0"

  backend "s3" {
    bucket = "your-bucket-name"  //e.g. terraform-state-bucket
    key    = "terraform.tfstate" //この文字列で tfstate ファイルが作られてS3に置かれる
    region = "ap-northeast-1"    //S3のリージョン
  }
}

provider "aws" {
  version = "~> 2.0"
  access_key = var.access_key // terraform v0.12 から "${}" をつけなくてよくなった
  secret_key = var.secret_key
  region     = var.region
}
```

上記のように `terraform.backend` を設定することで、State（ `*.tfstate` ）が S3 に保存される。（ `terraform init` 実行後）

## tfstate

tfstate は AWS への設定情報を保持し、現在の `.tf` ファイルとの差分を求めることによって AWS へ設定を反映させる。  
先の例では backend （ tfstate の保存場所）に S3 を設定したが、通常の場合はローカル（ `.terraform` ディレクトリ）に保存される。  
tfstate ファイルの保存場所は `terraform init` コマンド実行時の `.tf` ファイルの設定に基づいて作成される。  
複数人で開発する場合、個々人の tfstate に差分があっては、同じ `*.tf` ファイルであっても実行に差分が発生するため、 S3 など共有できる場所に tfstate を保存するのがよい。  
`backend` を設定せずに `terraform init` を実行すると以下のようになる。

```bash
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (terraform-providers/aws) 2.23.0...

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

$ ls -la
total 24
drwxr-xr-x  6 xxxx  staff  192  8 13 16:50 .
drwxr-xr-x  7 xxxx  staff  224  8 13 16:43 ..
drwxr-xr-x  3 xxxx  staff   96  8 13 16:50 .terraform <- できてる！
-rw-r--r--  1 xxxx  staff  140  8 13 16:48 aws.tf
-rw-r--r--  1 xxxx  staff   77  8 13 16:46 terraform.tfvars
-rw-r--r--  1 xxxx  staff   70  8 13 16:46 variables.tf
```

# 環境差分の扱い

独自なので参考までに。

```bash
$ tree
.
├── modules
│   ├── variables.tf # 変数定義、値の代入は各環境(dev/std/prd)の main.tf で
│   ├── aws.tf       # provider の設
│   ├── ec2.tf       # 以降、ここのリソース定義
│   └── その他もろもろ.tf
└── services
    ├── dev
    │   ├── main.tf
    │   └── terraform.tfvars # ローカルで access_key/secret_key だけ設定し、 git 管理はしない
    ├── std
    │   ├── main.tf
    │   └── terraform.tfvars
    └── prd
        ├── main.tf
        └── terraform.tfvars
```

各環境の `main.tf` は以下のような感じ。

```terraform
terraform {
  required_version = ">= 0.12"

  backend "s3" {
    bucket = "your-bucket-name" # backend では変数を使えないので注意
    key    = "terraform_dev.tfstate"
    region = "ap-northeast-1"
  }
}

variable "access_key" {}
variable "secret_key" {}
variable "region" {
  default = "ap-northeast-1"
}

module "my_module" {
  source  = "../../modules"
  access_key  = var.access_key
  secret_key  = var.secret_key
  region      = var.region
  // 以降、 modules の variables.tf 内の変数へ値を代入
  environment = "dev"
  hoge        = "hoge"
}
```

`terraform.tfvars` は以下のような感じ。

```terraform
access_key = "access_key"
secret_key = "secret_key"
```

`aws.tf` は以下。

```terraform
provider "aws" {
  version    = "~> 2.25"
  access_key = var.access_key
  secret_key = var.secret_key
  region     = var.region
}
```

`variable.tf` は以下のような感じ。

```terraform
variable "access_key" {}
variable "secret_key" {}
variable "region" {
  default = "ap-northeast-1"
}
variable "project" {
  default = "samplePrj"
}
variable "environment" {
  default = "dev"
}

locals {
  base_tags = {
    Project     = var.project
    Terraform   = "true"
    Environment = var.environment
  }

  base_name       = "${var.project}_${var.environment}"
  cluster_name    = "${local.base_name}_cluster"
  default_tags    = merge(local.base_tags, map("kubernetes.io/cluster/${local.cluster_name}", "shared"))
  cluster_version = "1.13"
}
```

各環境向けに実行するときは各環境のパス（ `/services/dev` など）をカレントにして実行する。

# 参考

- [Terraform Module Registry](https://registry.terraform.io/)
- [terraform-community-modules](https://github.com/terraform-community-modules)
- [AWS Provider](https://www.terraform.io/docs/providers/aws/index.html) を参照。
- [Terraform職人入門: 日々の運用で学んだ知見を淡々とまとめる](https://qiita.com/minamijoyo/items/1f57c62bed781ab8f4d7)
  - 環境差分の扱いなど参考
  - `module` と `-var-file` で環境差分（ `dev` `std` `prd` ）を扱うとよいと思った