---
title: "アーキテクチャ"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 2
---

**Kubernetes** （ **k8s** ）のアーキテクチャについて整理する。

- サーバ構成
- コンポーネント

<!--more-->

- [公式](https://kubernetes.io/ja/docs/home/)
- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
    - Kubernetesクラスタ構築のチュートリアルとして有名なドキュメント

## サーバ構成

k8s ではホストマシン（物理サーバ、もしくは仮想マシン、インスタンス）の種別として **Master Nodes** と **Worker Nodes** がある。  
それぞれで実行されるコンポーネントは以下の通り。各コンポーネントはそれぞれ各サーバ上で独立したプロセスとして常駐する。

- **Master Nodes** / **Control Plane** ： 管理サーバ。クライアントツール（ **kubectl** ）から Control Plane を経由して Worker Nodes をコントロールする。
    - `kube-apiserver`
    - `etcd`
    - `kube-controller-manager`
        - `node-controller` 、 `replication-controller`
    - `kube-scheduler`
    - `cloud-controller-manager`
- **Worker Nodes** / **Data Plane** ： Control Plane から管理されるワーカーサーバ。複数台で構成されるクラスタ構成をとり、コンテナがマウントされる。
    - `kubelet`
    - `kube-proxy`
    - `kube-dns` ・CoreDNS

### その他コンポーネント・クライアントツール

- その他のコンポーネント
    - `hyperkube`
        - k8s 関連のバイナリを1つにまとめたall-in-oneバイナリ
    - cAdvisor
        - 各 Node 上にあるコンテナのCPU、メモリ、ファイル、ネットワーク使用量といった、リソースの使用量と性能の指標を監視・収集するエージェント
        - `kubelet` に同梱されている
- クライアントツール
    - kubectl
        - k8s クラスタを管理するコマンド
    - kubeadm
        - 物理もしくは仮想サーバに対して Kubernetes 環境を構築するツール
    - kubefed
        - Federation を管理するコマンド
        - Federationは複数のリージョンやクラウドに配置されているKubernetesのクラスタを一括で管理できるようにする機能
    - Minikube
        - 1 台のローカル端末で シングル Node の Kubernetes Cluster をお試しできるツール
    - Dashboard
        - k8s クラスタの可視化を行う Web ベースのツール

## コンポーネント

ここでは先ほど紹介した Master Nodes / Worker Nodes に配備されている主要コンポーネントについて記載する。

### kube-apiserver / Master Nodes

Kubernetes API を外部に提供するコンポーネント。  
REST API を提供しているが、通常はクライアントツール `kubectl` でコントロールする。`kubectl` から設定情報を受け付けて、 クラスタが保持すべき状態として情報を `etcd` へ保存する。  
`kube-apiserver` 以外のコンポーネントは直接 `etcd` を参照せず、 `kube-apiserver` を通してリソースにアクセスする。  
なお、 `kube-apiserver` は水平スケールによりトラフィックを分散させることが可能。  
コンテナが作成されるまでのフロー例は以下のようになる。

1. `kubectl` で Deployment API を呼ぶ
    - `api-server` はリクエストの **認証** と **バリデーション** を実行し、 `etcd` にデータを保存したのち結果を返却
2. Deployment Controller が検知し、 ReplicaSet API を呼ぶ
    - `api-server` 経由で `etcd` に保存された Deployment に関する情報の変更を検知
3. ReplicaSet Controller が検知し、 Pod API を呼ぶ
    - `api-server` 経由で `etcd` に保存された ReplicaSet に関する情報の変更を検知
4. Scheduler が検知し、配置先 Node を決定し、 Pod API を呼ぶ
5. Kubelet が検知し、 Pod を作成
    - 各 Controller 、 Scheduler 、 Kubelet は常にそれぞれが Reconcilation Loop を回している

### etcd / Master Nodes

k8s クラスタの全ての情報が保存される分散 KVS 。  
`etcd` へアクセスするコンポーネントは `kube-apiserver` のみで、他のコンポーネントは `kube-apiserver` を介してアクセスする。  
分散合意アルゴリズム Raft の特性上、奇数台で冗長構成にする必要がある。  
`etcdctl shapshot` コマンドなどで定期的にバックアップしておく。

### kube-controller-manager / Master Nodes

5 秒の無限ループ周期で `kube-apiserver` 経由で `etcd` の情報とクラスタの現在の状態を比較し、 `etcd` の状態へ更新する（ `kube-apiserver` 経由）。
Controller は自作可能であるが、デフォルトで以下のような Controller がある。

- Node Controller
    - Worker Node 停止の検出と通知を担当する
- Deployment Controller
- Replication Controller
    - Pod の数を制御する
    - Replication Controller はリソース名自体に"Controller"とつくため、コントローラーは"RepplicationManager"という名前
- Endpoints Controller
    - Service と Pod の紐付けをを行、エンドポイントオブジェクトを設定する
- Service Account & Token Controllers
    - namespace のデフォルトアカウント、API アクセストークンを作成する

なお、 Controller は複数プロセス存在するが、バイナリとしては 1 つにまとめられる。
また、各 Controller は `go routine` で起動し、 `kube-apiserver` 経由で `etcd` へアクセスし、登録情報と現在のクラスタの状態を比較し続ける。  
`kube-controller-manager` は状態の差分を検出、 `etcd` へ登録するが、実際のクラスタへの反映作業は `kubelet` が行う。

### kube-scheduler / Master Nodes

`kube-scheduler` の仕事は、新規に `etcd` に登録された Pod に対して最適な Worker Nodes を選択して割り当てること。  
`go routine` の無限ループ（間隔 0s ）で以下の処理を繰り返す。

1. `kube-apiserver` 経由で `etcd` にアクセスし、Worker Nodes へ未割り当ての Pod 情報を 1 つ取得する
2. Pod の設定情報から最適な Woker Node を選択する
3. 選択した Worker Node の情報を`kube-apiserver` 経由で `etcd` にアクセスし、保存する

Worker Node の選択は、 `predicates` と呼ばれる条件にによるフィルタと、 `priorities` という優先度付けによって行われる。

- `predicates`
    - Worker Node をフィルタする条件。
    - 例えば nodeSelector による Worker Node の選択や request に対するリソースの空き状況によるフィルタがある。
- `priorities`
    - Worker Node の優先度付け。
    - 例えば Pod を出来る限り Worker Nodes、ゾーンを分散させるような優先度付けがある。

参考：https://qiita.com/tkusumi/items/58fdadbe4053812cb44e

### cloud-controller-manager / Master Nodes

クラウド特有の制御ロジックを組み込むコンポーネント。  
`kube-controller-manager` 同様、内部に以下のような Controller を持つ。

- Node Controller
    - Node が反応を停止した際、クラウドプロバイダー経由で Node 停止を検出する
- Routing Controller
    - クラウドインフラのルーティングを設定する
- Service Controller
    - クラウドプロバイダーのロードバランサの作成・更新・削除を行う

### kubelet / Worker Nodes

クラスタ内のそれぞれの Worker Node で稼働するエージェントで、実際にコンテナを起動・停止する。PodSpecs の設定に基づいて、 Pod 内のコンテナの稼働を管理する。  
`kube-scheduler` によって Pod が起動される Worker Node が決定した事を検知して処理を行う。  
また、 ConfigMap や Secret の設定更新は `kubelet` の Sync Loop （ `--node-status-update-frequency duration` というオプションがあり、デフォルト 10s ）のタイミングで反映される。  
`kubelet` は複数のコンテナランタイムを扱うことができ、以下のようなものがある。

- 高位コンテナランタイム / **CRIランタイム**： `kubelet` から利用されるランタイム
    - docker-shim (Docker)
        - なお、 docker-shim は CRIラインタイムではなく、 Docker Engine の ラッパー（ OCI から Docker API への変換）である（[参考１](https://blog.inductor.me/entry/2020/12/03/061329)、[参考２](https://thinkit.co.jp/article/18024)）
        - kubelet -> docker-shim -> Docker Engine -> OCIランタイム
    - [containerd](https://containerd.io/) （ Docker Engine 内部でも利用されている）
        - kubelet -> containerd -> OCIラインタイム ： containerd は CRI に準拠しており、 Docker Engine はしていない。なので docker-shim が必要になる。
    - [cri-o](https://cri-o.io/)
    - Frakti
    - rktlet (rkt)
    - 参考：[Worker Node へのCRIランタイム設定方法](https://kubernetes.io/ja/docs/setup/production-environment/container-runtimes/)
- 定位コンテナランタイム / **OCIランタイム**： 高位コンテナランライムから利用されるランタイム
    - runc
        - namespaces、cgroups、pivot_root、Linuxのセキュリティ機能などを利用して実装されている（詳しくは[ここ](https://medium.com/nttlabs/runc-overview-263b83164c98)）
    - runv
    - runsc (gVisor)
    - kata-runtaime ( Kata Containers )

上記のような複数のコンテナランタイムを利用できる背景には、 CRI ( [Container Runtime Interface](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) ) や OCI ( [Open Container Initiative](https://opencontainers.org/)) や CNI ( [Container Network Interface](https://github.com/containernetworking/cni) ) などの標準化が進んでいることが挙げられる。

また、 `kubelet` には `cAdvisor` が同梱されており、取得したメトリクスを `metrics-server` へ送信する。  
`metrics-server` はオンメモリにしかその結果を収集しないので、高級的なモニタリングを行うには他のソリューションを利用する必要がある。

### kube-proxy / Worker Nodes

k8s クラスタの各 Worker Node で動作するネットワークプロキシで、トラフィックのルーティングを担当する。  
`kube-apiserver` をポーリングして設定変更を検知する。  
ホスト上のネットワークルールを維持し、接続転送を実行することによって、Kubernetesサービスの抽象化を可能にしている。  
転送方式には以下の 3 つがあり、 `--proxy-mode` で指定できる。

- userspace mode
    - 受信したトラフィックを userspace 上のプロセスが処理する
- iptables mode
    - カーネルの iptable の nat table を設定して IP 変更やロードバランス（ClusterIP）の設定を行う
- ipvs mode

また、  Worker Node に跨った Pod 間での通信を可能にするため **オーバーレイネットワーク** を構築する必要がある。  
`kube-proxy` は以下のような [CNI](https://github.com/containernetworking/cni) Plugin と連携してオーバーレイネットワークを構築する。（詳しくは[ここ](https://ntoofu.github.io/blog/post/container-network-interface/)）

- Flannel
- Open vSwitch
- Project Calico
- Cilium
- Contiv (Cisco が開発？)
- Romana
- Weave Net

### kube-dns・CoreDNS / Worker Nodes

k8s クラスタの名前解決/**サービスディスカバリ**を担当するクラスタ内の DNS サーバ。  
`kube-apiserver` をポーリングして設定変更を検知する。  
`kube-dns` の後継として [CoreDNS](https://kubernetes.io/ja/docs/tasks/administer-cluster/coredns/) がある。

### metrics-server

```bash
$ git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
$ cd kubernetes-metrics-server/
$ kubectl apply -f .
```

なお、 `metrics-server` をインストールすることによって、 `kubectl top node` や `kubectl top pod` コマンドなどでメトリクス情報を確認できるようになる。  
`metrics-server` はオンメモリにのみメトリクス情報を保持するので、恒久的なモニタリングには他のソリューションが必要となる。