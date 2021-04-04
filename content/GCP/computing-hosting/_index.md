---
title: "コンピューティングとホスティング"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

1. Compute Engine

<!--more-->

## 1. Compute Engine

[公式ガイド](https://cloud.google.com/compute/docs/how-to?hl=ja)

**Compute Engine** は Google のインフラストラクチャ上に仮想マシンを構築するサービス。

1. [リージョンとゾーン](#1-リージョンとゾーン)
2. [インスタンス作成](#2-インスタンス作成)

（教材メモ：https://www.qwiklabs.com/focuses/3563?parent=catalog）

### 1.1. リージョンとゾーン

一部の Compute Engine リソースは、リージョン内またはゾーン内にのみ存在する。  
**リージョン** とはリソースを実行できる特定の地理的な場所で、各リージョンには、1 つまたは複数の **ゾーン**がある。  
たとえば、リージョン `us-central1` は米国中部を指し、ゾーン `us-central1-a`、`us-central1-b`、`us-central1-c`、`us-central1-f` が含まれる。  
（ GCP のゾーンは、 AWS でいう AZ：アベイラビリティゾーン）

ゾーン内で動作するリソースは **ゾーンリソース** と呼び、例えば、仮想マシンインスタンスと永続ディスクは 1 つのゾーン内で動作ゾーンリソース。
永続ディスクを仮想マシン インスタンスに接続するには、両方のリソースを同じゾーン内に配置する必要があり、同様に、インスタンスに静的 IP アドレスを割り当てるには、インスタンスが静的 IP と同じリージョンに存在している必要がある。

- [リージョンとゾーン](https://cloud.google.com/compute/docs/regions-zones/)

### 1.2. インスタンス作成

#### 1.2.1 ブートディスクイメージ

OS のブートディスクイメージには大きく以下の 2 種類ある。

- **公開 OS イメージ**
    - Google、オープンソース コミュニティ、サードパーティ ベンダーによって提供され、保守されるブートディスクイメージ
- **カスタムイメージ**
    - ユーザーが所有し、アクセスを制御するブートディスクイメージ
    - [カスタム イメージの作成、削除、廃止](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images?hl=ja)

公開 OS イメージの情報は、以下のコマンドで取得可能。

```zsh
% gcloud compute images list
NAME                              PROJECT       FAMILY         DEPRECATED  STATUS
centos-6-v20181011                centos-cloud  centos-6                   READY
centos-7-v20181011                centos-cloud  centos-7                   READY
coreos-alpha-1953-0-0-v20181106   coreos-cloud  coreos-alpha               READY
coreos-beta-1939-1-0-v20181106    coreos-cloud  coreos-beta                READY
coreos-stable-1911-3-0-v20181106  coreos-cloud  coreos-stable              READY
(略)
```

また、公開 OS イメージではなく、 自作の **カスタムイメージ** 

#### 1.2.2. インスタンス作成

```zsh
% gcloud compute instances create \
    VM_NAME \                                         # VM_NAME：インスタンス名
    [--image IMAGE | --image-family IMAGE_FAMILY] \   # IMAGE（公開イメージの必須バージョン） or IMAGE_FAMILY（イメージファミリ名）
                                                      # IMAGE_FAMILY を指定した場合、ファミリ内の非推奨でない最新の IMAGE が適用
    --image-project IMAGE_PROJECT                     # イメージプロジェクト名
```

上記コマンドで作成したら、下記コマンドで確認する。

```zsh
% gcloud compute instances describe VM_NAME
```

オプションで `--shielded-secure-boot` フラグを指定すると、以下の 3 つの [Shielded VM](https://cloud.google.com/security/shielded-cloud/shielded-vm?hl=ja) 機能をすべて有効にしたインスタンスを作成する。

- [仮想トラステッド プラットフォーム モジュール（vTPM）](https://cloud.google.com/security/shielded-cloud/shielded-vm?hl=ja#vtpm)
- [整合性モニタリング](https://cloud.google.com/security/shielded-cloud/shielded-vm?hl=ja#integrity-monitoring)
- [セキュアブート](https://cloud.google.com/security/shielded-cloud/shielded-vm?hl=ja#secure-boot)

インスタンスを起動した後、[Shielded VM のオプションを変更](https://cloud.google.com/compute/docs/instances/modifying-shielded-vm?hl=ja) するには、インスタンスを停止する必要がある。

また、プロジェクト毎に `default` という名前のデフォルト VPC ネットワークが作成されており、ネットワークの詳細を指定せずに VM を作成すると、デフォルト VPC ネットワークと同じリージョンにあるサブネットにインスタンスが作成される。  
以下のオプションを指定することで、対象のネットワークにインスタンスが作成される。

- `SUBNET_NAME`: サブネットの名前。ネットワークは指定したサブネットから推測される。
- `ZONE_NAME`: VM を作成するゾーン（ `europe-west1-b` など）。リージョンはこのゾーンから推測される。

### 1.3. プリエンプティブルインスタンス作成

#### 1.3.1. プリエンプティブルインスタンス

[プリエンプティブルインスタンス](https://cloud.google.com/compute/docs/instances/preemptible?hl=ja) は、通常より遥かに低価格である代わりに以下の制約をうけるインスタンス。

- システムイベントにより、いつでも停止する可能性がある
- 24 時間実行した後、必ず停止
- 有限の Compute Engine リソースなので、常に利用できるわけではない
- ライブ マイグレーションできない
- サービスレベル契約（ SLA ）の非対象（ [Compute Engine SLA](https://cloud.google.com/compute/sla?hl=ja) ）
- Google Cloud の無料枠に適用されない

#### 1.3.2. プリエンプティブルインスタンス作成

```zsh
% gcloud compute instances create [INSTANCE_NAME] --preemptible
```

### 1.4. 単一テナントノードにインスタンス作成

https://cloud.google.com/compute/docs/nodes/provisioning-sole-tenant-vms?hl=ja

#### 1.4.1. 単一テナントノード テンプレートを作成

[単一テナントノード](https://cloud.google.com/compute/docs/nodes/sole-tenant-nodes?hl=ja) のテンプレートを作成する。

```zsh
% gcloud compute sole-tenancy node-templates create TEMPLATE_NAME \
  --node-type=NODE_TYPE \
  [--node-affinity-labels=AFFINITY_LABELS \]
  [--region=REGION \]
  [--accelerator type=GPU_TYPE,count=GPU_COUNT \]
  [--disk type=local-ssd,count=DISK_COUNT,size=DISK_SIZE]
```

#### 1.4.2. ノードグループの自動スケーリング


```zsh
% gcloud compute sole-tenancy node-groups create group-name \
  --node-template template-name \
  --target-size size \
  --maintenance-policy maintenance-policy \
  --zone zone \
  --autoscaler-mode mode \
  --max-nodes max-nodes \
  --min-nodes min-nodes
```



#### メモ

- ノードテンプレート
    - ノードグループの各ノードのプロパティを定義するリージョン リソース
    - ノード テンプレートからノードグループを作成すると、ノード テンプレートのプロパティがノードグループの各ノードに不変にコピーされる
- ノードグループ
- ノードタイプ
- インスタンステンプレート
  - https://cloud.google.com/compute/docs/instance-templates?hl=ja


↓↓↓以降整理中↓↓↓

#### 1.2.1. Cloud Console から新しいインスタンスを作成する

Cloud Console のナビゲーションメニューで、「Compute Engine」-> 「VM インスタンス」 の順にクリック。（最初に初期化する際は 1 分ほどかかる）  
新しいインスタンスを作成するには、「作成」 をクリック。  
インスタンス作成時は、例えば、以下のようなパラメータを設定する。

|項目|設定例|その他の情報|
|:---|:---|:---|
|名前|gcelab|インスタンス名|
|リージョン|us-central1（アイオワ）||
|ゾーン|us-central1-c||
|シリーズ|N1|シリーズの名前|
|マシンタイプ|2 vCPU|n1-standard-2、2-CPU、7.5 GB RAM インスタンス|
|ブートディスク|新しい 10 GB の標準の永続ディスク OS イメージ: [Debian GNU/Linux 10 (buster)]||
|ファイアウォール|HTTP トラフィックを許可する|ポート 80 に HTTP トラフィックを許可するファイアウォールルール|

**マシンタイプ** は、マイクロインスタンスタイプから 32 コア、208 GB RAM のインスタンスタイプまで、選択できるマシンタイプは複数ある。（[詳細](https://cloud.google.com/compute/docs/machine-types)）  
**ブートディスク** には OS が設定されており、Debian、Ubuntu、CoreOS から、Red Hat Enterprise Linux や Windows Server などのプレミアム イメージまで、選択できるイメージは複数ある。

SSH を使用して仮想マシンに接続するには、該当するマシンの行で「SSH」をクリックする。

#### 1.2.2. gcloud で新しいインスタンスを作成する

Cloud Shell で、コマンドラインから gcloud を使用して新しい仮想マシン インスタンスを作成する。

```bash
$ gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-c

Created [...gcelab2].
NAME     ZONE           MACHINE_TYPE  ...    STATUS
gcelab2  us-central1-c  n1-standard-2 ...    RUNNING
```

上記のコマンドで作成されたインスタンスには以下のデフォルト値が設定される。

- 最新の Debian 10（buster）イメージ
- マシンタイプは n1-standard-2
- インスタンスと同じ名前のルート永続ディスクが自動的にインスタンスに組み込まれる

Cloud Console のナビゲーションメニューで、「Compute Engine」 -> 「VM インスタンス」の順にクリックし、インスタンスが作成されたか確認できる。  
また、 `gcloud` 経由で、SSH を使用してインスタンスに接続することもできる。（パスフレーズは空白）

```bash
$ gcloud compute ssh gcelab2 --zone us-central1-c
```