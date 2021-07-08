---
title: "概念整理"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 1
---

**Kubernetes** （ **k8s** ）の概念について整理する。

- k8s で実現できること
- k8s のリソース
- k8s の認証・認可・権限管理

<!--more-->

## k8s で実現できること

- 複数ホストにコンテナをデプロイ
- 関連するコンテナ毎にグルーピング
- コンテナの死活監視
- コンテナ間のネットワーク
- コンテナの負荷分散
- コンテナのリソース管理

## k8s のリソース

- Workloads リソース
    - コンテナの実行に関するリソース
    - https://thinkit.co.jp/article/13610
- Discovery＆LB リソース
    - コンテナを外部公開するようなエンドポイントを提供するリソース
    - https://thinkit.co.jp/article/13738
- Config＆Storage リソース
    - 設定・機密情報・永続化ボリュームなどに関するリソース
    - https://thinkit.co.jp/article/14139
- Cluster リソース
    - セキュリティやクォータなどに関するリソース
- Metadata リソース
    - リソースを操作する系統のリソース

### Workloads リソース

- Pod
    - 複数のコンテナ間で共有して使用する 1 つの仮想 NIC と、それを利用する複数のコンテナ（と Volume ）をまとめて Pod という
        - 複数のコンテナを 1 つの Pod にまとめることができ、 1 つの Pod で 1 つの仮想 NIC 、 Pod 内のコンテナで共有
    - k8s ではコンテナを起動する際、 Pod 単位で起動する
    - Pod 内の各コンテナは localhost の扱い
    - ReplicaSet が Pod 内のコンテナの多重度をコントロールする
        - 障害検知、オートスケール
    - Pod 内には複数種類のコンテナを格納でき、メインのコンテナに加えて、補助的な役割を担うコンテナ（サブコンテナ）を加える構成のことを **サイドカー** と呼ぶ
- ReplicaSet
    - ReplicationController （今後廃止）の後継
    - 複数の Pod を管理
    - Pod の レプリカを生成し、指定した数の Pod を維持し続けるリソース（ **セルフヒーリング** ）
    - 監視は、特定の **Label** がつけられた Pod の数をカウントする形で実現
        - レプリカ数が不足している場合は **template** から Pod を生成し、レプリカ数が過剰な場合は Label にマッチする Pod のうち1つを削除
    - selector をサポートする点において ReplicationController と異なる
    - set-based selector
    - ReplicaSet の特殊な形として「 DaemonSet 」「 StatefulSet 」がある
        - DaemonSet ：全て(またはいくつか)のNodeが単一のPodのコピーを稼働させることを保証する。各ノードに 1 Pod ずつ起動したい（Fluentdなど）ときに有効。
        - StatefulSet ：ReplicaSet がステートレスだとするとStatefulSetはステートフル。Podの識別子の順序管理など、個々の Pod の状態変化を前提として管理する。
- Deployments
    - Pod と ReplicaSet を一括で管理する ReplicaSet の上位互換
    - Deployment は複数の ReplicaSet を管理することで、ローリングアップデートやロールバックなどを実現可能にするリソース
    - `kubectl` でリソース管理する際、基本的には ReplicaSet を直接操作することはなく Deployment を操作する
- Job
    - コンテナを利用して一度限りの処理を実行させるリソース
- CronJob
    - ScheduledJob の後継
    - スケジュールされた時間に Job を生成
- DaemonSet
    - 各ノードに1つずつPodが起動することを保証
    - 例)ログエージェントPodにより各ノードのログを収集
- StatefulSet
    - ステートフルアプリケーションのデプロイに使用
    - データを永続ディスクストレージに保持
    - Pod IDに序数インデックスを使用(例:web-0、web-1、web-2)
- HorizontalPodAutoscaler (HPA)
    - 指定されたリソースターゲットを対象とし、レプリカ数を調整

Pod の管理・制御を行うリソース（オブジェクト）を **コントローラ** という。

### Discovery＆LB リソース

- Service
    - Pod の集合に対して外部と通信を行うための通り道、 1 つのマイクロサービスととらえることができる
    - ロードバランサのようなもので、 Pod へのアクセスをプロキシする
    - 「 Pod 宛トラフィックのロードバランシング」「サービスディスカバリと内部 DNS 」を実現
    - 「 IP + Port 」のアクセスを複数 Pod へ割り振る
    - Node へは Service の単位で配備される
- Ingress
    - 省略
- Endpoints

> ネットワークレイヤの復習
> - L1：物理レイヤ
> - L2：MAC アドレス
> - L3：IP アドレス
> - L4：TCP/UDP
> - L5：ソケット、セッション
> - L6：TLS
> - L7：HTTP

### Service

Service は以下の種類の **L4** ロードバランサを提供し、クラスタ内の各ノードに仮想敵に構成される。  
実質、ClusterIP、NodePort、LoadBalancer の 3 種類。

- **ClusterIP**
    - k8s **クラスタ内からのみ疎通可能な IP** となる Service （なので、「Cluster」IP）をノード内に仮想敵に構成する
    - k8s クラスタ外から通信を受け付ける必要のない箇所のロードバランサ
    - 各 Pod は IP を持つが個々にアクセスしていては負荷分散できないのでそれを 1 つの IP に束ねる
    - name がホスト名として機能し、 クラスタ内部 DNS により名前解決される
        - `<name>.<namespace>.svc.cluster.local`
        - `svc` は Service の略
    - 各ノードの `kube-proxy` 通信の転送を行う
    - type は `ClusterIP`
    - 設定する Port は以下の通り
        - `port` ： Service のポート。 Service は内部的な仮想 IP アドレスを持っており、 Pod へ転送する際の From のポート。
        - `targetPort` ：転送先の Pod のボート。
- ExternalIP（ ClusterIP の一種）
    - 特定のノードの IP アドレスで受信した通信をコンテナに転送する Service
    - k8s クラスタ外部からの通信を受け付ける
    - type 自体は `ClusterIP` で、 `spec.externalIPs` （ノードのIP）を指定する
    - `spec.externalIPs` の IP は自分でノードの IP を調べて記載する必要があり、 `spec.ports[].port` の port で受け、該当セレクタの `spec.ports[].targetPort` へ流す
- **NodePort**
    - クラスタ内の各ノードに外部からアクセス可能な IP を与え、ノード内の仮想的な Service にトラフィックを転送する
    - ExternalIP に類似したサービスだが、ノードの IP を指定する必要がなく、全ノードが対応する
    - 違いは「ノードのIPアドレス:ポート」で通信を受信する点
        - 厳密には「0.0.0.0:ポート」でバインドされ、k8s クラスタ内の全ノードの IP アドレスを意味する
    - デフォルトで利用できるノードポートの範囲は「30000〜32767」であり、クラスタ内でユニークなポートでなければならないことに注意
    - type は `NodePort`
    - 設定する Port は以下の通り
        - `nodePort` ：ノードが受け取るポート。このポートが受けたリクエストは Service へ転送される。
        - `port` ： Service のポート。 Service は内部的な仮想 IP アドレスを持っており、 Pod へ転送する際の From のポート。
        - `targetPort` ：転送先の Pod のボート。
- **LoadBalancer**
    - 商用環境で k8s クラスタ外部から通信を受ける際に良い Service
    - k8s クラスタ外部（例えばロードバランサ）から疎通性のある仮想 IP を払い出せる
    - ExternalIP や NodePort と異なり、ノードIP非依存である点で使い勝手がよい
    - type は `LoadBalancer`
    - クラウド利用の場合、サービスからのトラフィックをクラスタ内のノードに転送するクラウドプロバイダが提供するロードバランサが実態
        - そのため、基盤が LoadBalancer Service に対応している必要がある
    - 設定する Port は以下の通り
        - `nodePort` ：ノードが受け取るポート。このポートが受けたリクエストは Service へ転送される。
        - `port` ： Service のポート。 Service は内部的な仮想 IP アドレスを持っており、 Pod へ転送する際の From のポート。
        - `targetPort` ：転送先の Pod のボート。
- Headless（None）
    - ロードバランシングする仮想 IP アドレスが払い出されない DNS ラウンドロビンのエンドポイントを提供する Service
    - type 自体は `ClusterIP` だが、 `clusterIP` が `None`
- ExternalName
    - k8s クラスタ内部から外部へアクセスするための Service
- None-Selector
    - ？？

Service の転送先は Pod となり（ Deployment/Replicaset ではない）、その Pod の `labels` を `selector` で指定することにより転送先を特定する。
Service の `selector` の設定が正しく Pod を捉えているかどうかは `kubectl get pods --selector="app=monolith,secure=enabled"` などのコマンドで対象の Pod を取得できるか、で検査できる。

### Ingress

Service とは異なり、 **L7** ロードバランサを提供する。  
`kind: Service` ではなく、 `kind: Ingress` で提供される。  
大きく以下の種類がある。

- k8s クラスタ外の LB を利用した Ingress
    - ex. GKE Ingress Controller
- k8s クラスタ内に Ingress 用の Pod をデプロイする Ingress
    - ex. Nginx Ingress Controller

Ingress はその実態である GKE/Nginx Ingress Controller とその設定である Ingress Resource からなる。  
Ingress は L7 とあるように、例えば gRPC のロードバランシングの際に必要となり、構成は以下のようになる。

```
例）
  grpc client
   ↓
  AWS ELB(classic)
   ↓
  kubernetes(service)
   ↓
  nginx-ingress(on k8s) ...  -> Ingress Resouce の設定を参照する
  ↓                 ↓
grpc server1    grpc server2
```

### Config＆Storage リソース

コンテナに対して設定ファイル、パスワードなどの機密情報などをインジェクトしたり、永続化ボリュームを提供したりするためのリソース。  
Kubernetesでは、個別のコンテナに対する設定の内容は環境変数やファイルが置かれた領域をマウントして渡すことが一般的

- ConfigMap
    - 単純な Key-Value の設定を参照する場合に利用
- Secret
    - 機密情報（ID、Passwordなど）を含む環境変数を参照する場合に利用
    - マニフェスト上で秘匿化部分は base64 化されているだけなので、暗号化したい場合は `kubesec` などを利用する
- PersistentVolume (PV)
    - クラスタ内の永続ストレージの管理に使用
    - GCPでの標準はCompute Engineの永続ディスク
- PersistentVolumeClaim (PVC)
    - PVへのリクエスト
    - PVへの具体的なサイズ、アクセスモード、StorageClass をリクエストする
    - PodはPVC通じてボリュームを使用。リクエストを満たすPVが存在しプロビジョニング可能な場合、PVCはPVにバインドされる

Config＆Storage リソースとまとめてはいるが、 ConfigMap や Secret もコンテナ間でデータを共有できる Volume と言える。

## k8s の認証・認可・権限管理

- [Kubernetesのユーザー管理と認証・権限確認機構を理解しよう](https://knowledge.sakura.ad.jp/21129/)