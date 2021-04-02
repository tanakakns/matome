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

次に、NGINX ウェブサーバーをインストールする  
SSH ターミナルで、root アクセス権を取得。

```bash
$ sudo su -
```

root ユーザーとして、OS を更新。

```bash
$ apt-get update
```

NGINX をインストール。

```bash
$ apt-get install nginx -y
```

NGINX が実行されていることを確認。

```bash
$ ps auwx | grep nginx
```

ウェブページを表示するために、Cloud Console に戻って該当するマシンの行の 「外部 IP」（ External IP ） のリンクをクリックする。

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