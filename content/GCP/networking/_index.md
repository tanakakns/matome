---
title: "ネットワーキング"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 7
---

1. コンセプト
2. VPC
3. Cloud Load Balancing

<!--more-->

- [Google Cloud ネットワーキング プロダクト](https://cloud.google.com/products/networking)
- 接続
    - [Virtual Private Cloud（VPC）](https://cloud.google.com/vpc/docs)

## 1. コンセプト

- VPC ネットワーク
    - グローバルリソース、特定のリージョンやゾーンに関連付けできない
    - リージョンを跨げる、IP レンジは指定しない
    - VPC 内であれば、異なるリージョンのサブネット同士でプライベート IP で通信可能
    - 異なる VPC に所属するサブネット同士はプライベート IP で通信できないため、インターネット経由/Cloud VPN/VPCピアリングなどを利用する
- サブネット
    - リージョンリソース、サブネット毎に IP アドレスの範囲が定義される
    - ゾーンを跨げる
- ファイアウォールルール
    - VM インスタンスで発生するトラフィックをルールベースで許可・拒否
    - 上り（インスタンスに向かっていくトラフィック）・下り（インスタンスから出ていくトラフィック）それぞれで設定する
    - タグやサービスアカウントを適用条件に加えることもできる
    - 以下の暗黙のルールがデフォルトで適用されている（ Cloud Console に表示されない）
        - 暗黙の下り許可ルール：すべてのトラフィックを許可（アクション： `allow` 、宛先： `0.0.0.0/0` ）
        - 暗黙の上り許可ルール：すべてのトラフィックを許可（アクション： `deny` 、送信元： `0.0.0.0/0` ）

## 2. VPC

Virtual Private Cloud（VPC）は、Compute Engine 仮想マシン（VM）インスタンス、Google Kubernetes Engine（GKE）クラスタ、[App Engine フレキシブル環境](https://cloud.google.com/appengine/docs/flexible) にネットワーキング機能を提供する。

### 2.1. VPC ネットワーク

- VPC ネットワークは、データセンター内のリージョン仮想サブネットワーク（ **サブネット** ）のリストで構成されるグローバルリソースであり、すべてグローバルな広域ネットワークで接続される。
- **ルート**、**ファイアウォール** ルールを含む VPC ネットワークは **グローバルリソース** 。特定のリージョンやゾーンには関連付けられていない。
- サブネットは [リージョンリソース](https://cloud.google.com/compute/docs/regions-zones/global-regional-zonal-resources#regionalresources) 。サブネットごとに IP アドレスの範囲が定義される。（ VPC には範囲が定義されない）

### 2.2. VPC のオプション

VPC のオプション、相互接続には以下のサービスがある。

|サービス|内容|
|:---|:---|
|共有 VPC |組織内の複数プロジェクトから共通の VPC として利用できる|
|VPC ピアリング|異なる VPC 間でピアリングを形成し、組織・プロジェクトが異なる VPC であっても互いのサブネット同士がプライベート IP で通信できる|
| Cloud VPN | IPSec 接続を使用して VPC 同士、またはオンプレミスと VPC を接続する |
| Cloud Interconnect |オンプレミスと VPC を専用線接続し、低レイテンシかつ高可用の接続を実現|

### 2.3. サブネット作成モード

サブネット作成モードには、自動モードとカスタムモードの 2 種類ある。

- 自動モード
    - すべてのリージョンに 1 つずつサブネットが自動的に作成
    - 事前定義された IP 範囲（ `10.128.0.0/9` ）が使用される
    - GCP に新しくリージョンが追加されると、サブネットも自動的に追加される
    - カスタムモードへ変更可能
- カスタムモード
    - すべてユーザが作成する必要がある
    - サブネットの IP 範囲は作成後に変更可能であるが、VPC 内の他のサブネットと範囲が重複しないようにする必要があり、且つ、縮小は不可能

### 2.4. ネットワークの作成

Cloud Console では、ナビゲーション メニュー（mainmenu.png） > [VPC ネットワーク] > [VPC ネットワーク]から作成する。  
コマンドでの VPC 作成は以下。（自動モードではなくカスタムモードを指定）

```bash
$ gcloud compute networks create privatenet --subnet-mode=custom
```

先に作成した VPC にサブネットを 2 つ追加する。

```bash
$ gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24

$ gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west4 --range=172.20.0.0/20
```

作成したら、以下のコマンドで VPC およびサブネットを確認する。

```bash
$ gcloud compute networks list
NAME           SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
default        AUTO         REGIONAL
managementnet  CUSTOM       REGIONAL
mynetwork      AUTO         REGIONAL
privatenet     CUSTOM       REGIONAL

$ gcloud compute networks subnets list --sort-by=NETWORK
NAME                 REGION                   NETWORK        RANGE
（省略）
privatesubnet-us     us-central1              privatenet     172.16.0.0/24
privatesubnet-eu     europe-west4             privatenet     172.20.0.0/20
```

ファイアウォールは、Cloud Console で、ナビゲーション メニュー（mainmenu.png） > [VPC ネットワーク] > [ファイアウォール] から作成する。  
コマンドでは以下。（先ほど作成した VPC に追加）

```bash
$ gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0

Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-ed0a2d7fdd5c/global/firewalls/privatenet-allow-icmp-ssh-rdp].
Creating firewall...done.
NAME                           NETWORK     DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
privatenet-allow-icmp-ssh-rdp  privatenet  INGRESS    1000      icmp,tcp:22,tcp:3389        False
```

作成したファイアウォールは以下のコマンドで確認する。

```
$ gcloud compute firewall-rules list --sort-by=NETWORK
NAME                              NETWORK        DIRECTION  PRIORITY  ALLOW                         DENY  DISABLED
（省略）
privatenet-allow-icmp-ssh-rdp     privatenet     INGRESS    1000      icmp,tcp:22,tcp:3389                False
```

これまで作成した VPC、サブネット に VM インスタンスを作成するコマンドは以下。

```bash
$ gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatesubnet-us
```

作成したら VM インスタンスの存在を確認する。

```bash
$ gcloud compute instances list --sort-by=ZONE
NAME                 ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
（省略）
privatenet-us-vm     us-central1-c   n1-standard-1               172.16.0.2   35.223.37.245   RUNNING
```


## 3. Cloud Load Balancing

### 3.1. 種類

Cloud Load Balancing の負荷分散の種類には以下がある。

|種類|内部/外部|リージョン/グローバル|プロキシ/パススルー|Docs|補足|
|:---|:---|:---|:---|:---|:---|
|内部 TCP/UDP |内部|リージョン限定|パススルー|[内部 TCP/UDP 負荷分散の概要](https://cloud.google.com/load-balancing/docs/internal)||
|内部 HTTP(S) |内部|リージョン限定|プロキシ|[内部 HTTP(S) 負荷分散](https://cloud.google.com/load-balancing/docs/l7-internal)||
| TCP/UDP ネットワーク|外部|リージョン限定|パススルー|[外部 TCP / UDP ネットワーク負荷分散](https://cloud.google.com/load-balancing/docs/network)|ネットワークロードバランサ|
| TCP プロキシ|外部|グローバル|プロキシ|[TCP プロキシ負荷分散](https://cloud.google.com/load-balancing/docs/tcp)||
| SSL プロキシ|外部|グローバル|プロキシ|[SSL プロキシ負荷分散](https://cloud.google.com/load-balancing/docs/ssl)||
| 外部 HTTP(S) |外部|グローバル|プロキシ|[外部 HTTP(S) 負荷分散](https://cloud.google.com/load-balancing/docs/https)|HTTP(S) ロードバランサ|

- 内部/外部
    - 外部：インターネットからのトラフィック
    - 内部： VPC 内のトラフィック
- リージョン/グローバル
    - グローバル
        - バックエンドが複数リージョンに分散
        - 単一のエニーキャスト IP アドレスを使用してアクセスを提供（ IPv6 もある）
        - 「外部」かつ「プロキシ」の負荷分散
    - リージョン
        - バックエンドが同一リージョン内
        - IPv4 のみ
        - 「内部」または「パススルー」の負荷分散
- プロキシ/パススルー
    - プロキシ（ NAT 型）
        - LB で一度接続終端する（送信元 IP が LB の内部 IP に変わる）ため、ファイアウォールルールは VPC ネットワーク内を許可する必要がある
        - バックエンド VM からのレスポンスは、ロードバランサを経由して、クライアントに送信される
    - パススルー（ Direct Server Return : DSR 型）
        - 接続終端しないのでクライアントの IP がそのまま届くため、ファイアウォールルールは `0.0.0.0/0` からのトラフィックを許可する必要がある
        - バックエンド VM からのレスポンスは、ロードバランサを経由せず、クライアントに直接送信される（DSR）

選択方法としては以下のようになる。

<img src="img/choose-lb.svg" />

ネットワークロードバランサ と HTTP(S) ロードバランサ の特徴は以下。

- ネットワークロードバランサ
    - バックエンド サービスベースのネットワーク ロードバランサ（非レガシー、ただしプレビュー版）
        - バックエンド サービスを使うと、レガシー ヘルスチェックではサポートされていない新しい機能が有効になる
        - 非レガシー ヘルスチェック（TCP、SSL、HTTP、HTTPS、HTTP/2）のサポート、マネージド インスタンス グループによる自動スケーリング、コネクション ドレイン、構成可能なフェイルオーバー ポリシーなどが可能
    - ターゲット プールベースのネットワーク ロードバランサ（レガシー）
        - 以前は、ネットワーク ロードバランサの唯一の選択肢
        - ターゲット プールは、Google Cloud のネットワーク ロードバランサでサポートされているレガシー バックエンド
        - ターゲット プールは、ロードバランサから受信トラフィックを受け取るインスタンスのグループを定義している
- HTTP(S) ロードバランサ
    - 複数のバックエンド タイプをサポート
        - インスタンス グループ
        - ゾーン ネットワーク エンドポイント グループ（NEG）
        - サーバーレス NEG: 1 つ以上の App Engine、Cloud Run、Cloud Functions サービス
        - インターネット NEG: Google Cloud の外部にあるエンドポイント（カスタム送信元とも呼ばれる）
        - Cloud Storage のバケット

HTTP(S) ロードバランサ の構成イメージは以下。

<img src="img/https-load-balancer-diagram.svg" />

### 3.2. 作成

#### 3.2.1. ネットワークロードバランサのコンセプト

- ターゲットプール
- 静的外部 IP アドレス
- レガシー HTTP ヘルスチェック リソース
- 転送ルール

#### 3.2.2. ネットワークロードバランサの作成

デフォルトのリージョンとゾーンを設定する。

```bash
$ gcloud config set compute/zone us-central1-a
$ gcloud config set compute/region us-central1
```

3 つの Web サーバを構築する。（インスタンス名のみ異なり、それぞれ metadata の startup-script で apache をインストール・起動している。）

```bash
$ gcloud compute instances create www1 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"

$ gcloud compute instances create www2 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www2</h1></body></html>' | tee /var/www/html/index.html"

$ gcloud compute instances create www3 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www3</h1></body></html>' | tee /var/www/html/index.html"
```

上記のインスタンスに適用するファイアウォール（ `network-lb-tag` がついたインスタンスに対して `80` ポートアクセスを許可）を作成する。

```bash
$ gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80

Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/firewalls/www-firewall-network-lb].
Creating firewall...done.
NAME                     NETWORK  DIRECTION  PRIORITY  ALLOW   DENY  DISABLED
www-firewall-network-lb  default  INGRESS    1000      tcp:80        False
```

インスタンス一覧から「 EXTERNAL_IP 」を確認して、それぞれ `curl` で疎通する。

```bash
$ gcloud compute instances list
NAME  ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
www1  us-central1-a  n1-standard-1               10.128.0.2   35.238.246.91   RUNNING
www2  us-central1-a  n1-standard-1               10.128.0.3   35.232.249.208  RUNNING
www3  us-central1-a  n1-standard-1               10.128.0.4   34.123.247.106  RUNNING

$ curl http://35.238.246.91
!doctype html><html><body><h1>www1</h1></body></html>

$ curl http://35.232.249.208
!doctype html><html><body><h1>www2</h1></body></html>

$ curl http://34.123.247.106
!doctype html><html><body><h1>www3</h1></body></html>
```

ここまでが、負荷分散対象となる Web サーバ群の作成。

ターゲット プールベースのネットワーク ロードバランサを作成して上記 3 つの Web サーバを適用する。  
まずは、静的外部 IP アドレスを作成する。

```bash
$ gcloud compute addresses create network-lb-ip-1 \
 --region us-central1

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/regions/us-central1/addresses/network-lb-ip-1].
```

次に、レガシー HTTP ヘルスチェック リソースを作成する。

```bash
$ gcloud compute http-health-checks create basic-check

NAME         HOST  PORT  REQUEST_PATH
basic-check        80    /
```

次に、インスタンスと同じリージョンに **ターゲットプール** を追加する。  
ターゲットプールが機能するにはヘルスチェックが必要なため、先ほど作成した HTTP ヘルスチェックリソースを適用して作成する。

```bash
$ 
gcloud compute target-pools create www-pool \
    --region us-central1 --http-health-check basic-check

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/regions/us-central1/targetPools/www-pool].
NAME      REGION       SESSION_AFFINITY  BACKUP  HEALTH_CHECKS
www-pool  us-central1  NONE                      basic-check
```

ターゲットプールに 3 つの Web サーバインスタンスを追加する。

```bash
$ gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3

Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/regions/us-central1/targetPools/www-pool].
```

静的外部 IP アドレスに来たトラフィックを ターゲットプールに転送する **転送ルール** を作成する。

```bash
$ gcloud compute forwarding-rules create www-rule \
    --region us-central1 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/regions/us-central1/forwardingRules/www-rule].
```

これで、静的外部 IP にアクセスすれば、 3 つの Web サーバに負荷分散される。

```bash
$ gcloud compute addresses list
NAME             ADDRESS/RANGE  TYPE      PURPOSE  NETWORK  REGION       SUBNET  STATUS
network-lb-ip-1  35.232.6.132   EXTERNAL                    us-central1          IN_USE # IP 確認

$ gcloud compute forwarding-rules describe www-rule --region us-central1
IPAddress: 35.232.6.132 # IP 確認
IPProtocol: TCP
creationTimestamp: '2021-04-07T05:45:21.364-07:00'
description: ''
fingerprint: NinUJkv44qA=
id: '3582287157978470286'
kind: compute#forwardingRule
labelFingerprint: 42WmSpB8rSM=
loadBalancingScheme: EXTERNAL
name: www-rule
networkTier: PREMIUM
portRange: 80-80
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/regions/us-central1
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/regions/us-central1/forwardingRules/www-rule
target: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/regions/us-central1/targetPools/www-pool

$ curl http://35.232.6.132

# 連続で負荷分散の様子を確認
$ while true; do curl -m1 35.232.6.132; done
# www1/2/3 が不規則に出る
```

#### 3.2.3. HTTP(S) ロードバランサのコンセプト

- インスタンステンプレート
    - VM インスタンス や マネージドインスタンスグループ（ MIG ）を作成するために使用できるリソース
    - マシンタイプ、ブートディスク イメージまたはコンテナ イメージ、ラベル、その他のインスタンス プロパティを定義できる
- [マネージドインスタンスグループ（ MIG ）](https://cloud.google.com/compute/docs/instance-groups?hl=ja)
    - 単一のエンティティとして管理できる VM インスタンスのグループ
    - 自動スケーリング、自動修復、リージョン（マルチゾーン）デプロイメント、自動更新などの自動化 MIG サービスを活用できる
    - 負荷分散のためヘルスチェック ではなく、 マネージド インスタンス グループのヘルスチェック があり、自動修復の基準となる
    - `gcloud compute instance-groups managed create`
- グローバル静的外部 IP アドレス
- ヘルスチェエク
- バックエンドサービス
- URL マップ
- ターゲット HTTP プロキシ
- グローバル転送ルール

#### 3.2.4. HTTP(S) ロードバランサの作成

負荷分散ターゲットの基となる インスタンステーンプレート　を作成する。

```bash
$ gcloud compute instance-templates create lb-backend-template \
   --region=us-central1 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --image-family=debian-9 \
   --image-project=debian-cloud \
   --metadata=startup-script='#! /bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/instanceTemplates/lb-backend-template].
NAME                 MACHINE_TYPE   PREEMPTIBLE  CREATION_TIMESTAMP
lb-backend-template  n1-standard-1               2021-04-07T06:01:46.131-07:00
```

作成したテンプレートに基づいて、マネージドインスタンスグループ（ MIG ）を作成する。

```bash
$ gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=us-central1-a

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/zones/us-central1-a/instanceGroupManagers/lb-backend-group].
NAME              LOCATION       SCOPE  BASE_INSTANCE_NAME  SIZE  TARGET_SIZE  INSTANCE_TEMPLATE    AUTOSCALED
lb-backend-group  us-central1-a  zone   lb-backend-group    0     2            lb-backend-template  no
```

`fw-allow-health-check` ファイアウォール ルールを作成する。  
これは、Google Cloud ヘルスチェック システム（130.211.0.0/22 と 35.191.0.0/16）からのトラフィックを許可する上り（内向き）ルール。  
ターゲットタグ `allow-health-check` を使用して VM が識別される。

```bash
$ gcloud compute firewall-rules create fw-allow-health-check \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80

Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/firewalls/fw-allow-health-check].
Creating firewall...done.
NAME                   NETWORK  DIRECTION  PRIORITY  ALLOW   DENY  DISABLED
fw-allow-health-check  default  INGRESS    1000      tcp:80        False
```

ロードバランサにユーザーが接続する際に使用するグローバル静的外部 IP アドレスを作成する。

```bash
$ gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --global

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/addresses/lb-ipv4-1].    

# 作成した IP のアドレスを確認
$ gcloud compute addresses describe lb-ipv4-1 \
>     --format="get(address)" \
>     --global
34.120.128.63
```

ロードバランサのヘルスチェックを作成。

```bash
$ gcloud compute health-checks create http http-basic-check --port 80

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/healthChecks/http-basic-check].
NAME              PROTOCOL
http-basic-check  HTTP
```

ヘルスチェックを適用した **バックエンドサービス** を作成。

```bash
$ gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/backendServices/web-backend-service].
NAME                 BACKENDS  PROTOCOL
web-backend-service            HTTP
```

MIG をバックエンドとしてバックエンドサービスに追加。

```bash
$ gcloud compute backend-services add-backend web-backend-service \
    --instance-group=lb-backend-group \
    --instance-group-zone=us-central1-a \
    --global

Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/backendServices/web-backend-service].
```

デフォルトの バックエンドサービス に受信リクエストをルーティングする URL マップを作成。

```bash
$ gcloud compute url-maps create web-map-http \
    --default-service web-backend-service

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/urlMaps/web-map-http].
NAME          DEFAULT_SERVICE
web-map-http  backendServices/web-backend-service
```

作成した URL マップにリクエストをルーティングするターゲット HTTP プロキシを作成。

```bash
$ gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/targetHttpProxies/http-lb-proxy].
NAME           URL_MAP
http-lb-proxy  web-map-http
```

受信リクエストをプロキシにルーティングするグローバル転送ルールを作成。

```bash
$ gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4-1\
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-aa3df922e690/global/forwardingRules/http-content-rule].
```

Cloud Console のナビゲーション メニューで、[ネットワーク サービス] > [Cloud Load Balancing] に移動して、作成したロードバランサ（web-map-http）をクリック。  
[バックエンド] セクションでバックエンドの名前をクリックして、VM が正常であることを確認。  
ブラウザで「http://34.120.128.63」にアクセスして動確（リロードしまくり）。「lb-backend-group-xxxx」の「xxxx」が変わることで、 MIG 内の別インスタンスに負荷分散されていることがわかる。