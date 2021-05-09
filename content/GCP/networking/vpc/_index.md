---
title: "VPC"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

<!--more-->

- [Virtual Private Cloud（VPC）](https://cloud.google.com/vpc/docs)

## 1. コンセプト

- VPC ネットワーク
    - グローバルリソース、特定のリージョンやゾーンに関連付けできない
    - リージョンを跨げる、IP レンジは指定しない
    - VPC 内であれば、異なるリージョンのサブネット同士でプライベート IP で通信可能
    - 異なる VPC に所属するサブネット同士はプライベート IP で通信できないため、インターネット経由/Cloud VPN/VPCピアリングなどを利用する
- サブネット
    - リージョンリソース（マルチゾーンリソース）
    - サブネット毎に プライマリおよびセカンダリ CIDR 範囲（IP アドレスの範囲）を定義できる
    - VM インスタンスは、プライマリ CIDR 範囲からプライマリ内部 IP アドレスを取得する
    - また、プライマリおよびセカンダリ CIDR 範囲に [エイリアス IP 範囲](https://cloud.google.com/vpc/docs/alias-ip?hl=ja) を定義できる
        - **IP エイリアス** は、物理的に一つの NIC に 2つ以上の IP アドレスを割り当てる方法で、 エイリアス IP 範囲は文字通りその範囲
        - この技術を利用することで、あるクライアントが サーバ A の IP エイリアスに接続している状況で、その指定先を変更せずに サーバ B に同じ IP エイリアスを切り替えることができる
    - なお、VM インスタンスのネットワーク インターフェースには、1 つのプライマリ内部 IP アドレスが必要であり、1 つ以上のエイリアス IP 範囲、1 つの外部 IP アドレスを設定できる
- IP アドレス
    - 内部 IP アドレス
        - インターネット通信不可能（インターネット通信するにはなんらかの 外部 IP アドレスを持つプロキシを介する必要がある）
    - 外部 IP アドレス
        - インターネット通信可能
        - 静的外部 IP アドレスには、リージョンまたはグローバルリソースを指定できる
        - グローバル静的外部 IP アドレスは、グローバル負荷分散に使用されるグローバル転送ルールでのみ使用可能
    - 内部・外部 IP アドレスは、エフェメラル（動的）または静的のいずれかになる
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