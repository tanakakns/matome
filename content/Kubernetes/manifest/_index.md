---
title: "マニフェスト"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 5
---

kubernetes では `kubectl` に様々コマンドがあるが、基本的には `kubectl create/delete/apply` コマンドと **マニフェスト** とよぶ YAML/JSON ファイルを使用することで、リソースの作成/削除/更新できる。

1. [マニフェストの雛形](#1-マニフェスト雛形)
2. [マニフェストの理解](#2-マニフェスト理解)
3. その他のリソース

<!--more-->

- [Kubernetes リファレンス](https://kubernetes.io/ja/docs/reference/)
- [マニフェスト リファリンス](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/)

## 1. マニフェストの雛形

ドライラン（ `kubectl run/create --dry-run=client` ）と yaml 表示（ `-o yaml` / `--output=yaml` を）組み合わせるとマニフェストの雛形を標準出力できる。

```yaml
$ kubectl run sample --image nginx -o yaml --dry-run=client
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: sample
  name: sample
spec:
  replicas: 1
  selector:
    matchLabels:
      run: sample
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: sample
    spec:
      containers:
      - image: nginx
        name: sample
        resources: {}
status: {}
```

- deployment
    - `kubectl run mydeploy --image=nginx --output=yaml --dry-run=client`
- pod
    - `--restart=Never` を付けると Pod になる
    - `kubectl run mypod --restart=Never --image=nginx --output=yaml --dry-run=client`
- job
    - `--restart=OnFailure` を付けると Job になる
    - `kubectl run myjob --restart=OnFailure --image=ubuntu --output=yaml --dry-run=client -- echo hello`
- cronjob
    - `--schedule` を付けると CronJob になる
    - `kubectl run mycron --schedule "1 * * * *" --image=nginx --output=yaml --dry-run=client`
- service
    - `kubectl create svc clusterip myapp --tcp=80 --output=yaml --dry-run=client`
- configmap
    - `kubectl create cm mycm --from-literal mykey=myval --output=yaml --dry-run=client`
    - `--from-file` でファイルを指定した場合ちゃんとインデントしてくれる
    - `kubectl create cm mycm --from-file myfile.yaml --output=yaml --dry-run=client`
- secret
    - 値は base64 エンコードされているので編集に注意
    - `kubectl create secret generic mysecret --from-literal mykey=myval --output=yaml --dry-run=client`
- serviceaccount
    - `kubectl create serviceaccount mysc --output=yaml --dry-run=client`
- clusterrolebinding
    - `kubectl create clusterrolebinding myclusterrolebinding --clusterrole=edit --serviceaccount default:mysc --output=yaml --dry-run=client`
- rolebinding
    - `kubectl create rolebinding cluster-admin-binding --role=edit --serviceaccount default:mysc --output=yaml --dry-run=client`
- poddisruptionbudget
    - `kubectl create pdb my-pdb --selector=app=nginx --min-available=1 --output=yaml --dry-run=client`

## 2. マニフェストの理解

``` yaml:sample-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: nginx
      imege: nginx:1.7.9
      ports:
        - containerPort: 80
```

全てのマニフェストには以下の項目が含まれる。

- apiVersion
    - API のバージョン
- kind
    - リソースの種類
- metadata
    - リソースのメタデータ

Pod の場合は `spec` 以下に Pod の内容を定義する。

```bash
$ kubectl create -f sample-pod.yaml
pod "sample-pod" created

$ kubectl apply -f sample-pod.yaml
pod "sample-pod" configured

$ kubectl delete -f sample-pod.yaml
pod "sample-pod" deleted
```

`kubectl apply` は **マニフェストに変更がある場合のみ変更処理を行う** 。  
削除済みのマニフェストに対しても差分を検出するため、新規作成でも `apply` を使用するほうがよい。  
また、マニフェストには複数のリソースを同時に記述することができ、 **`---`** で区切る。

```yaml:sample-pod.yml
---
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: nginx
      imege: nginx:1.7.9
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
# LBの追加など
```

また、複数のマニフェストファイルを一気に適用したい場合は、ディレクトリにまとめ `-f` で指定して実行することができる。  
その際、ファイル名辞書順で実行されることに注意。

### 2.1. Label と Annotation

両者とも key-value 形式（ `key: value` ）でデータを保持する仕組みであるが、用途が異なる。

#### 2.1.1. Label

- 各リソースの `metadata` で設定する
- あるリソースが 1 つ以上のリソースを管理する場合、 `selector` にて Label を指定することにより対象リソースを発見する
  - ReplicaSet が Pod を管理する場合
  - Deployment が ReplicaSet を管理する場合
  - Service が受けた通信の転送先 Pod を指定する場合、など

#### 2.1.2. Annotation

- リソースオブジェクトに関する不特定の情報を保存する方法
  - オブジェクトの変更理由の記録
  - 特別なスケジューラへの、特別なスケジュールポリシーの伝達
  - リソースを更新したツールと、それがどのように更新したかの情報の付加
  - 他のツールからの更新検知などのため
  - Deployment オブジェクトによるロールアウトのための、ReplicaSet の追跡情報の保存、など

### 2.2. namespace

namespace は k8s クラスタ上で論理的にシステムを分割する仕組み。  
namespace リソース（ `kind: Namespace` ）を作成し、各リソースの `metadata` にラベル付け（ `namespace: test` ）して利用する。

### 2.3. Deployment

リソースには色々あるが、理解すべきリソースは少ない。そのうちの一つ。  
（その他は、Service、DaemonSet、CronJobくらい？）

```yaml
apiVersion: app/v1
kind: Deployment
metadata: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#objectmeta-v1-meta
  annotations:                # 任意のメタデータを保存および取得する。外部ツールなどによって設定・変更・保存される箇所でもある。
    key: value
  clusterName: string         # オブジェクトが所属するクラスタ名。異なるクラスターで同じ名前と名前空間を持つリソースを区別するために使用。使わない。
  creationTimestamp:          # このオブジェクトが作成されたサーバ時間。変更できない。
  deletionGracePeriodSeconds: # オブジェクトの正常終了までの時間。読み取り専用。
  deletionTimestamp:          # オブジェクトが削除される時間。読み取り専用。
  finalizers:                 # リストからエントリを削除する責任のあるコンポーネントの識別子。削除のみ可能。
  generateName:               # name が設定されていない場合にのみオプションで設定される名前。
  generation:                 # 状態の世代を表す。読み取り専用。
  initializers:               # オブジェクトの作成時にシステムの不変条件を強制するコントローラー。DEPRECATED。
  labels:                     # ReplicaSet、Service の selector で利用。
    key: value
  managedFields:              # 内部のワークフロー用でユーザは設定・理解の必要はない。
  name:                       # namespace 内で一意なオブジェクトの名前。
  namespace:                  # 名前空間名。空は default と同じ。 DNS_LABEL である必要があり、変更できない。
  ownerReferences:            # このオブジェクトに依存するオブジェクトのリスト。コントローラにより管理される。
  resourceVersion:            # オブジェクトがいつ変更されたかを判断するためにクライアントが使用できるこのオブジェクトの内部バージョン。読み取り専用。
  selfLink:                   # このオブジェクトを表すURL。読み取り専用。
  uid:                        # このオブジェクトの一意な ID 。システムにより管理される。
spec: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#deploymentspec-v1-apps
  minReadySeconds:         # コンテナがクラッシュすることなく、新しく作成されたポッドの準備が整うまでの最小秒数。デフォルト 0 。
  paused: bool             # 展開が一時停止されていることを示す。
  progressDeadlineSeconds: # 展開が失敗したと見なされるまで最大秒数。
  replicas:                # Pod のリプリケーション数。
  revisionHistoryLimit:    # ロールバックを許可するReplicaSetの世代数。デフォルト 10 。
  selector: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#labelselector-v1-meta
            # ReplicaSet が Pod を選択するための Label Selector 。
    matchExpressions: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#labelselectorrequirement-v1-meta
                      # Label Selector のリスト。 AND で評価される。
    matchLabels:      # key-value 形式の Label Selector のリスト。 AND で評価される。matchExpressions の方が複雑な表現が可能。
  strategy: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#deploymentstrategy-v1-apps
            # 新しいポッドに置き換えるため展開戦略。現状は単純な再配置とローリングアップデートのみ。
    rollingUpdate: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#rollingupdatedeployment-v1-apps
      maxSurge:       # Pod のレプリケーション数を超えて作成できるコンテナの最大数。数か割合で表現。
      maxUnavailable: # Pod のレプリケーション数を下回って利用できなくする最大数。数か割合で表現。
    type: # Recreate か RollingUpdate 。デフォルト RollingUpdate 。
  template: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#podtemplatespec-v1-core
            # 作成する Pod の定義
    metadata: # 先の metadata に同じ。
    spec: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#podspec-v1-core
      activeDeadlineSeconds: # Pod が非アクティブになってから強制終了されるまでの秒数。
      affinity: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#affinity-v1-core
                # Pod をスケジューリングする際の制約。
      automountServiceAccountToken: # サービスアカウントトークンを自動的にマウントする必要があるかどうかを示す。
      containers: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#container-v1-core
                  # Pod に属するコンテナのリスト。
        args:                     # ENTRYPOINT へ引数。
        command:                  # ENTRYPOINT 配列。
        env:                      # 環境変数のリスト。
        envFrom:                  # 環境変数を設定するソースのリスト。
        image:                    # Docker イメージ名。
        imagePullPolicy:          # Docker イメージの pull ポリシー。 Always, Never, IfNotPresent のいずれか。
        lifecycle:                # コンテナのライフサイクルイベントに応じて管理システムが実行するアクション。
        livenessProbe:            # コンテナ死活の定期的な調査。
        name:                     # コンテナ名。DNS_LABEL となる。
        ports:                    # コンテナの公開ポートのリスト。
        readinessProbe:           # コンテナサービスの準備状況の定期的な調査。
        resources:                # コンテナが利用するコンピュートリソース。
        securityContext:          # Pod を実行するセキュリティオプション。
        stdin:                    # コンテナがコンテナーランタイムで標準入力にバッファーを割り当てるかどうか。デフォルト false は EOF になる。
        stdinOnce:                # 単一の接続によって開かれた後に、コンテナランタイムがstdinチャネルを閉じるかどうか。デフォルト false 。
        terminationMessagePath:   # オプション：コンテナの終了メッセージが書き込まれるファイルがコンテナのファイルシステムにマウントされるパス。
        terminationMessagePolicy: # 終了メッセージの入力方法。
        tty:                      # コンテナに TTY を割り当てるか否か。デフォルト false 。
        volumeDevices:            # コンテナが使用するブロックデバイスのリスト。
        volumeMounts:             # コンテナのファイルシステムにマウントするポッドボリューム。
        workingDir:               # コンテナの作業ディレクトリ。
      dnsConfig: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#poddnsconfig-v1-core
        nameservers: # DNSネームサーバーのIPアドレスのリスト。
        options:     # DNSリゾルバーオプションのリスト。
        searches:    # ホスト名検索用のDNS検索ドメインのリスト。
      dnsPolicy:   # Pod の DNS ポリシー。ClusterFirstWithHostNet, ClusterFirst（デフォルト）, Default, None 。
      enableServiceLinks: # サービスに関する情報をDockerリンクの構文と一致する Pod の環境変数に注入するか否か。デフォルト true 。
      hostAliases: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#hostalias-v1-core
                   # Pod の hosts ファイルへの追加設定。
      hostIPC:     # ホストの IPC 名前空間を使うか否か。デフォルト false 。
      hostNetwork: # Pod に要求されたホストネットワーキング。ホストのネットワーク名前空間を使用。デフォルト false 。
      hostPID:     # ホストの PID 名前空間を利用するか否か。デフォルト false 。
      hostname:    # Pod のホスト名。指定しない場合システムが付与。
      imagePullSecrets: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#localobjectreference-v1-core
                        # Docker イメージを Pull する際のシークレット。
      initContainers:    # Pod に属する初期化コンテナのリスト。コンテナが開始される前に順番に実行される。
      nodeName:          # Pod を特定のノードへスケジューリングする。
      nodeSelector:      # Pod を配置するノードのセレクタ。
      overhead:          # 特定の RuntimeClass の Pod の実行に関連するリソースオーバーヘッドを表す。コントローラにより自動入力される。
      preemptionPolicy:  # 優先度の低い Pod を先取りするポリシー。
      priority:          # 優先度の値。値が大きいほど優先度が高い。
      priorityClassName: # Pod の優先順位を示す。
      readinessGates: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#podreadinessgate-v1-core
                      # Pod の準備状況を評価する。
      restartPolicy:    # Pod 内の全てのコンテナの再起動ポリシー。
      runtimeClassName: # Pod 実行に使用する RuntimeClass オブジェクト。
      schedulerName:    # Pod を管理するスケジューラ。
      securityContext: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#podsecuritycontext-v1-core
                       # Pod レベルのセキュリティ属性。
      serviceAccount:                # Deprecated
      serviceAccountName:            # Pod の実行に使用する ServiceAccount 名。
      shareProcessNamespace:         # Pod 内の全てのコンテナ間で PID 空間を共有する。デフォルト false 。
      subdomain:                     # サブドメインを指定する。"<hostname>.<subdomain>.<pod namespace>.svc.<cluster domain>" 。
      terminationGracePeriodSeconds: # Pod が Graceful Stop するのに要する秒数。デフォルト 30 秒。
      tolerations: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#toleration-v1-core
                   # Pod を特定のノードに配置する際に利用。 nodeSelector の方が古い。
      volumes: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#volume-v1-core
               # マウントできるボリュームのリスト。
```

上記、すごい量だが、必要最低限は。

```yaml
apiVersion: app/v1
kind: Deployment
metadata:
  annotations:
    <key>: <value>
  labels:                  # Deployment の Label
    <key>: <value>
  name: <string>           # Deployment 名
  namespace: <string>      # Deployment の namespace
spec:
  replicas: <integer>      # ReplicaSet のレプリカ数。
  selector:
    matchLabels:
      <key1>: <value1>     # ReplicaSet の対象セレクタ。以下の Pod 定義 の Label に一致させる。
  template:                # Pod 定義
    metadata:
      labels:              # Pod の Label
        <key1>: <value1>
    spec:
      containers:
        image: <string>    # Pod に利用するコンテナイメージ
        name: <string>     # Pod 名
        ports:
        imagePullPolicy: <string> # Docker イメージの pull ポリシー。 Always（常にpull）, Never（ローカルのみ）, IfNotPresent（ローカルに無ければpull）
        livenessProbe:            # コンテナ死活の定期的な調査。
        readinessProbe:           # コンテナサービスの準備状況の定期的な調査。
        resources: {}
```

ケースによっては追加で以下を意識する必要がある。

- spec.strategy
- spec.template.spec.affinity : [参考](https://qiita.com/sheepland/items/ed12b3dc4a8f1df7c4ec)

`livenessProbe`、`readinessProbe` については [こちら](https://kubernetes.io/ja/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) 。

### 2.4. Pod

```yaml
apiVersion: core/v1
kind: Pod
metadata: # 先の metadata に同じ。
spec:     # 先の Deployment の spec.template.spec に同じ。
status:   # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#podstatus-v1-core
```

pod のマニフェストに以下を足したものが ReplicaSet 。

```yaml
spec:
  replicas: <integer>
  selector:
    matchLabels:
      <key1>: <value1>
  template:
      <podの定義に同じ>
```

なお、 ReplicaSet はマニフェスト上はほぼ Deployment に同じ

#### 2.4.1. Lifecycle / Lifecycle Events / Lifecycle Handler

Pod には [のライフサイクル](https://kubernetes.io/ja/docs/concepts/workloads/pods/pod-lifecycle/) がり、 `status.phase` は Pod のライフサイクルにおけるフェーズを表しており、 Pod の状態を確認する上で重要となる。

|値|概要|
|:---|:---|
|Pending|PodがKubernetesシステムによって承認されが、コンテナの作成が完了していない状態。スケジュール・イメージダウンロード中などの状態。|
|Running|PodがNodeにバインドされ、少なくとも 1 つのコンテナが作成され、開始・実行・再起動中の状態。|
|Succeeded|JobなどのPod内のすべてのコンテナが正常に終了し、再起動も発生していない状態。|
|Failed|Pod内のすべてのコンテナが終了し、少なくとも1つのコンテナが異常終了した状態。コンテナはゼロ以外のステータスで終了したか、システムによって終了された状態。|
|Unknown|何らかの理由により、通常はPodのホストとの通信にエラーが発生したために、Podの状態を取得できなかった。|

上記のようなフェーズを決定するために、 Kubernetes では以下の 2 種類の **ヘルスチェック** がある。

- Liveness Probe
  - Pod が正常に動作しているかの確認
  - 失敗した場合、 Pod は再起動される
- Readiness Probe
  - Pod がサービスインする準備が出来ているかの確認
  - 失敗した場合、 Pod にトラフィックが流れず、再起動されることもない（サービスインされるまで待つ）

上記の設定はデフォルトではされておらず、通常は Service/LoadBalancer から Node への ICMP による簡易的なチェックしか実施されていない。  
そのため、 Pod の状態に応じたサービスの健全性を保つために上記の設定が必要になる。  
Liveness/Readiness Probe の双方で以下の 3 種類のヘルスチェックを設定できる。  
ヘルスチェックは kubelet からコンテナ毎に行われ、どれか 1 つのコンテナでも失敗した場合には Pod 全体が失敗とみなされる。

- exec
  - コマンドを実行し、終了コードが 0 でなければ失敗
  - 例
  ```
  livenessProbe:
    exec:
      command: ["ls", "/usr/sbin/nginx"]
  ```
- httpGet
  - HTTP GET リクエストを実行し、 Status Code が 200 〜 300 でなければ失敗
  - 例
  ```
  libenessProbe:
    httpGet:
      path: /health
      port: 80
      scheme: HTTP
      host: web.example.com
      httpHeaders:
      - name: Authorization
        value: Bearer TOKEN
  ```
- tcpSocket
  - TCP セッションが確立できなければ失敗
  - 例
  ```
  linenessProbe:
    tcpSocket:
      port: 80
  ```

また、コンテナが開始された際とPodが削除される際にHookして任意の処理を実行することが出来る以下の Lifecycle Events がある。

- postStart
    - コンテナが作成された後、即座に実行される
    - このHandlerが失敗した場合終了させ、 restartPolicy に従い再起動させる
- preStop
    - コンテナが終了される前に実行される
    - コンテナのルートプロセスに対して SIGTERM が送出される前にこのHandlerが実行される

postStart や preStop に指定できる Lifecycle Handler の種類は以下の通り。

- exec : コンテナ内でコマンドを実行
- httpGet : HTTPのGETリクエストを発行

先の probe と同様だ。

#### 2.4.2. Resource Requirements

Pod に割り当てるリソースの要求について記載する。

```yaml
resources:
  requests:
    cpu: 100m
    memory: 64Mi
  limits:
    cpu: 2
    memory: 1Gi
```

`resuests` は下限、 `limits` は上限と理解すればいい。

### 2.5. Service

```yaml
apiVersion: v1
kind: Service
metadata: # 先の metadata に同じ。
spec: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#servicespec-v1-core
  clusterIP:
  externalIPs:
  externalName:
  externalTrafficPolicy:
  healthCheckNodePort:
  loadBalancerIP:
  loadBalancerSourceRanges:
  ports:
  publishNotReadyAddresses:
  selector:
  sessionAffinity:
  sessionAffinityConfig:
  type: # ExternalName 、 ClusterIP （デフォルト）、 NodePort 、および LoadBalancer 。
```

色々設定があるが、 ClusterIP の場合は以下くらいでいい。

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    <key>: <value>
  labels:             # Service の Label
    <key>: <value>
  name: <string>      # Service 名
  namespace: <string> # Service の namespace
spec:
  selector:
    <key>: <value>    # 通信を流す Pod の Label をセレクタで指定
  ports:
  - protocol: TCP     # Service が受信するプロトコルとポートマッピング
    port: 80          # Service が受信 Port
    targetPort: 9376  # Pod の Port
```

## 3. その他のリソース

基本的なリソース以外に以下について記載する。

- [HPA（HorizontalPodAutoscaler）](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [PBD（PodDisruptionBudget）](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)

### 3.1. HPA（HorizontalPodAutoscaler）

HPA は pod の負荷に応じて自動的に pod の数を水平スケールさせるリソース。  
Deployment, ReplicaSet, ReplicationController, StatefulSetはscaled resource objectと呼ばれ、これらがauto scalingの対象リソースとなる。

```apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: php-hpa
  namespace: default
spec:
  scaleTargetRef: # ここでautoscale対象となる`scaled resource object`を指定
    apiVersion: apps/v1
    kind: Deployment
    name: php-deploy
  minReplicas: 1 # 最小レプリカ数
  maxReplicas: 5 # 最大レプリカ数
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50 # CPU使用率が常に50%になるように指定
```

[参考](https://qiita.com/sheepland/items/37ea0b77df9a4b4c9d80)

### 3.2. PBD（PodDisruptionBudget）

- `kubectl drain` によって対象 Node から Pod を退去でき、以降も Pod がスケジューリングされないように出来る
  - Node のメンテナンス(reboot)などを行う必要がある場合に kubectl drainを行うと Node で動いている Pod について gracefully に terminate される。また、ReplicaSet を使っていれば別の Node で Pod が自動的に起動される。メンテナンス完了後、kubectl uncordonを行うことで再度 Pod がスケジューリングされる状態になる。
- 注意点として上記だけでは対象 Node 上で起動している Pod は一度に evicted され、一度に退去させられる可能性がある。例えば ReplicaSet 2 で drain 対象の Node に 2つの Pod が起動していた場合、両方の Pod に対して evicted され、一つも Pod が起動していない時間帯がある可能性が出来てしまう
- PodDisruptionBudget(PDB)を定義することで上記を防ぐことが出来る

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: myapp
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
run: myapp
```

[参考](https://qiita.com/toshihirock/items/5adaece0206127121551)

## 参考

- [Kubernetes Tips: kubectl でマニフェストの雛形を作る](https://qiita.com/tkusumi/items/4f63cea4c4d910b368c4)
- [Kubernetes流YAML職人になるために](https://qiita.com/bgpat/items/173f33ae2a9e21b24487)