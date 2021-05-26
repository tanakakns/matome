---
title: "Google Kubernetes Engine"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 2
---

<!--more-->

[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/how-to)

1. [コンセプト](#1-コンセプト)
2. [クラスタ作成](#2-クラスタ作成)
3. [デプロイ管理](#3-デプロイ管理)
4. [スケール](#4-スケール)
5. [監視](#5-監視)
6. [ユースケース](#6-ユースケース)

## 1. コンセプト

ToDo

## 2. クラスタ作成

デフォルトのコンピューティング ゾーンを設定する。

```bash
$ gcloud config set compute/zone us-central1-a

Updated property [compute/zone].
```

GKE クラスタを作成する。

```bash
# USAGE
$ gcloud container clusters create [CLUSTER-NAME]

# 具体例
$ gcloud container clusters create my-cluster
Creating cluster my-cluster in us-central1-a... Cluster is being health-checked (master is healthy)...done.
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-47d564cd39d1/zones/us-central1-a/clusters/my-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-a/my-cluster?project=qwiklabs-gcp-01-47d564cd39d1
kubeconfig entry generated for my-cluster.
NAME        LOCATION       MASTER_VERSION   MASTER_IP      MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
my-cluster  us-central1-a  1.18.16-gke.302  34.67.190.125  e2-medium     1.18.16-gke.302  3          RUNNING
```

クラスタの認証情報を取得する。（ kubeconfig に設定が追加される）

```bash
# USAGE
$ gcloud container clusters get-credentials [CLUSTER-NAME]

# 具体例
$ gcloud container clusters get-credentials my-cluster
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-cluster.
```

試しに作成したクラスタにコンテナをデプロイする。

```bash
$ kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
deployment.apps/hello-server created

$ kubectl expose deployment hello-server --type=LoadBalancer --port 8080
service/hello-server exposed

$ kubectl get service
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
hello-server   LoadBalancer   10.3.245.136   35.238.58.233   8080:30372/TCP   70s
kubernetes     ClusterIP      10.3.240.1     <none>          443/TCP          10m

$ curl http://35.238.58.233:8080
Hello, world!
Version: 1.0.0
Hostname: hello-server-57684579f-kw54q
```

クラスタを削除する。

```bash
# USAGE
$ gcloud container clusters delete [CLUSTER-NAME]

# 具体例
$ gcloud container clusters delete my-cluster
The following clusters will be deleted.
 - [my-cluster] in [us-central1-a]
Do you want to continue (Y/n)?  Y
Deleting cluster my-cluster...done.
Deleted [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-47d564cd39d1/zones/us-central1-a/clusters/my-cluster].
```

### 2.1. 限定公開 Kubernetes クラスタのセットアップ

Kubernetes Engine における限定公開クラスタとは、公共のインターネットからマスターにアクセスできないようにするクラスタ。  
このクラスタのノードにはプライベート アドレスしかない（パブリック IP アドレスがない）ため、隔離された環境でワークロードが実行される。  
ノードとマスターは、VPC ピアリングを使用して相互に通信する。  
Kubernetes Engine API では、アドレス範囲が CIDR（Classless Inter-Domain Routing）ブロックとして表される。

```bash
$ gcloud beta container clusters create private-cluster \
    --private-cluster \
    --master-ipv4-cidr 172.16.0.16/28 \
    --enable-ip-alias \
    --create-subnetwork ""
```

なお、カスタム サブネットワークを使用する限定公開クラスタの作成は以下。

```bash
# サブネットワークとセカンダリ範囲を作成
$ gcloud compute networks subnets create my-subnet \
    --network default \
    --range 10.0.4.0/22 \
    --enable-private-ip-google-access \
    --region us-central1 \
    --secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14
# サブネットワークを使用する限定公開クラスタを作成
$ gcloud beta container clusters create private-cluster2 \
    --private-cluster \
    --enable-ip-alias \
    --master-ipv4-cidr 172.16.0.32/28 \
    --subnetwork my-subnet \
    --services-secondary-range-name my-svc-range \
    --cluster-secondary-range-name my-pod-range

# 外部アドレス範囲を承認（プライベートアクセスできる、IP の CIDR を設定する）
# [MY_EXTERNAL_RANGE] は、前の出力で取得した外部アドレスの CIDR 範囲に置き換え
$ gcloud container clusters update private-cluster2 \
    --enable-master-authorized-networks \
    --master-authorized-networks [MY_EXTERNAL_RANGE]
```

## 3. デプロイ管理

### 3.1. Deployment オブジェクトの詳細

`kubectl explain` コマンドを使用すると、Deployment などのオブジェクトの設定項目に関する情報が取得できる。

```bash
$ kubectl explain deployment

# 全てのフィールドを確認する場合
$ kubectl explain deployment --recursive

# 特定のフィールドを確認する場合
$ kubectl explain deployment.metadata.name
```

### 3.2. Deployment の作成

Google 提供の資材を使って、Cloud Console の Cloud Shell 上で実施していく。  
まずはクラスタを作成する。

```bash
# デフォルトのゾーンを設定
$ gcloud config set compute/zone us-central1-a

# サンプルコードを入手
$ gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
$ cd orchestrate-with-kubernetes/kubernetes

# 5 つの n1-standard-1 ノードを含むクラスタを作成
$ gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"

NAME      LOCATION       MASTER_VERSION   MASTER_IP     MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
bootcamp  us-central1-a  1.18.16-gke.302  35.238.203.7  e2-medium     1.18.16-gke.302  5          RUNNING
```

`deployments/auth.yaml` の `image` を以下のように変更する。

```yaml
…
containers:
- name: auth
  image: kelseyhightower/auth:1.0.0
...
```

Deployment でレプリカが 1 つ作成され、バージョン 1.0.0 の auth コンテナが使用される。  
`kubectl create` を使用して Deployment オブジェクトを作成する。

```bash
$ kubectl create -f deployments/auth.yaml

# 確認
$ kubectl get deployments
$ kubectl get replicasets
$ kubectl get pods
```

次に、auth デプロイ用のサービスを作成する。

```bash
$ kubectl create -f services/auth.yaml
```

ここまでで `auth` を作成したが、 `hello` 、 `frontend` も作成していく。

```bash
# hello
$ kubectl create -f deployments/hello.yaml
$ kubectl create -f services/hello.yaml

# frontend
$ kubectl create secret generic tls-certs --from-file tls/
$ kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
$ kubectl create -f deployments/frontend.yaml
$ kubectl create -f services/frontend.yaml
```

`frontend` の外部 IP を確認して `curl` で動確する。

```bash
$ kubectl get services frontend
NAME       TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)         AGE
frontend   LoadBalancer   10.115.243.177   34.66.110.111   443:31848/TCP   46s

$ curl -ks https://34.66.110.111
{"message":"Hello"}
```

### 3.3. スケールアウト・イン

Deployment をスケーリングするには、 `spec.replicas` フィールドを更新する。  
`kubectl explain` コマンドで、このフィールドの説明を確認してみる。

```bash
$ kubectl explain deployment.spec.replicas
KIND:     Deployment
VERSION:  apps/v1
FIELD:    replicas <integer>
DESCRIPTION:
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.
```

`kubectl scale` コマンドでスケールアウト・インする。

```bash
# Pod 数を 5 へスケールアウト
$ kubectl scale deployment hello --replicas=5

# 確認
$ kubectl get pods | grep hello- | wc -l
5

# Pod 数を 3 へスケールイン
$ kubectl scale deployment hello --replicas=3

# 確認
$ kubectl get pods | grep hello- | wc -l
3
```

### 3.3. ローリング アップデート

ReplicaSet が新たに作成され、古い ReplicaSet のレプリカ数が減少するにつれて、新しい ReplicaSet のレプリカ数が徐々に増加する形で実行される。  
Deployment を更新するには、次のコマンドを実行してマニフェストを編集する。

```bash
$ kubectl edit deployment hello
# image を「kelseyhightower/hello:1.0.0」から「kelseyhightower/hello:2.0.0」へ変更
```

新しい ReplicaSet の確認、および、ロールアウト履歴にも新しいエントリがあるか確認する。

```bash
$ kubectl get replicaset
NAME                  DESIRED   CURRENT   READY   AGE
auth-7cffdb8677       1         1         1       14m
frontend-7b4b97c4dc   1         1         1       12m
hello-687bb4b8f4      0         0         0       13m
hello-d99c798f        3         3         3       60s

$ kubectl rollout history deployment/hello
deployment.apps/hello
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

ロールアウトの状況は以下のコマンドで確認できる。

```bash
$ kubectl rollout status deployment/hello

# もしくは

$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

ロールアウトの状況が芳しくない場合、以下のコマンドで停止・再開ができる。


```bash
# ロールアウト停止
$ kubectl rollout pause deployment/hello

# ロールアウト再開
$ kubectl rollout resume deployment/hello
```

なお、なんらか問題が発生して、ロールバックが必要な場合は以下のコマンドで実施できる。

```bash
# ロールバック
$ kubectl rollout undo deployment/hello

# ロールアウト履歴確認
$ kubectl rollout history deployment/hello
deployment.apps/hello
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

# マニフェストを確認（2.0.0 -> 1.0.0 に戻ってる）
$ kubectl edit deployment hello

# 全ての Pod がもどっているか確認
$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

### 3.4. カナリア デプロイ

カナリア デプロイでは、一部のユーザーに変更をリリースすることで、新しいリリースに伴うリスクを軽減する手法。  
新しいバージョンの Deployment を作成する。

```bash
$ kubectl create -f deployments/hello-canary.yaml

# 確認
$ kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
auth           1/1     1            1           28m
frontend       1/1     1            1           26m
hello          3/3     3            3           27m
hello-canary   0/1     1            0           8s  # 新しい hello
```

この状態だと、単純に新しい別の Deployment を作成しただけに見えるが、 hello-canary の Label に `app:hello` が設定されており、これは hello service のセレクタになっているため、リクエストがルーティングされ、カナリアデプロイされた状態になっている。  
以下のコマンドを何度が実行していると、レスポンスが異なっており、カナリアデプロイされていることがわかる。

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
# {"version":"1.0.0"} or {"version":"2.0.0"}
```

ユーザーをいずれかのデプロイに「固定」したい場合、 **セッション アフィニティ** を指定してサービスを作成することで実現できる。  
以下の例では Client の IP でルーティングを固定している。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  sessionAffinity: ClientIP
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
```

### 3.5. Blue / Green デプロイ

Kubernetes は、古い「Blue」バージョン用と新しい「Green」バージョン用に 2 つの異なるデプロイを作成することでこれを実現します。  
このデプロイには、ルータとして機能するサービスを介してアクセスする。  
新しい「Green」バージョンが稼働したら、サービスを更新してそのバージョンを使用するように切り替えることにより実現する。

既存の hello サービスを、`app:hello` 、 `version: 1.0.0` のセレクタを持つように更新する。（元々は `app:hello` だけ）

```bash
$ kubectl apply -f services/hello-blue.yaml
# resource service/hello is missing という警告は無視
```

Green の Deployment をデプロイする。（元々の hello は Blue の扱いで image のバージョンは 1.0.0）

```bash
$ kubectl create -f deployments/hello-green.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
```

現状は Blue （つまり 1.0.0）であることを確認する。

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
# 何度叩いても 1.0.0
```

次に新しいバージョン（つまり Green ）を指定するようにサービスを更新する。（ セレクタが `version: 2.0.0` になっている）

```bash
$ kubectl apply -f services/hello-green.yaml
```

`curl` で確認すると今度は `2.0.0` しか返ってこない。（つまり、 Blue -> Green になった！）

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

ロールバックしたいときは再度 service を Blue に戻せばいい。（つまり、セレクタを `version: 1.0.0` にするだけ）

```bash
$ kubectl apply -f services/hello-blue.yaml

# 確認
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
# 1.0.0 しか返ってこない
```

## 4. スケール

- [クラスタオートスケーラー](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler)
    - GKE スラスタのノードプールサイズを自動変更することにより実現（ MIG ではないことに注意！）
    - `gcloud container clusters create` コマンドの `--enable-autoscaling --min-nodes 1 --max-nodes 4` オプションで指定
- [水平 Pod 自動スケーリング](https://cloud.google.com/kubernetes-engine/docs/concepts/horizontalpodautoscaler)
    - HPA ( [Horizontal Pod Autoscaler](https://kubernetes.io/ja/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) ) により実現される
- [垂直 Pod 自動スケーリング](https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler)
    - カスタムリソース`VerticalPodAutoscaler` により実現される

## 5. 監視

### 5.1. Cloud Monitoring と Cloud Logging

- GKE 用 Google Cloud オペレーション スイートの概要
    - https://cloud.google.com/stackdriver/docs/solutions/gke?hl=ja#skm-howto

GKEには、Cloud Monitoring および Cloud Logging との統合機能が含まれている。（ [Prometheus の使用](https://cloud.google.com/stackdriver/docs/solutions/gke/prometheus?hl=ja) はこちらを参照）  
`gcloud container clusters create` コマンドでクラスタを作成した場合、デフォルトで有効になっているが、既存のクラスタで有効にする場合は以下のコマンドを実行する。

```bash
$ gcloud container clusters update [CLUSTER_NAME] --enable-stackdriver-kubernetes
# ちなみに削除は「-no-enable-stackdriver-kubernetes」オプション
```

なお、現在はベータ版であるが、「 `--enable-logging-monitoring-system-only` 」オプションでシステムログのみ収集可能になる。（デフォルトは全ログ）  
システムログとは以下を指す。

- 名前空間 `kube-system` 、 `istio-system` 、 `knative-serving` 、 `gke-system` 、 `config-management-system`  で実行中のすべての Pod
- コンテナ化されていない重要なサービス:  `docker/containerd` ランタイム、 `kubelet` 、 `kubelet-monitor` 、 `node-problem-detector` 、 `kube-container-runtime-monitor`
- ノードのシリアルポート出力（VM インスタンスのメタデータ `serial-port-logging-enable` が true に設定されている場合）

アプリケーションログは、 コンテナが STDOUT と STDERR に書き込まれたワークロードのログを収集する。

[GKE ログの管理](http://cloud.google.com/stackdriver/docs/solutions/gke/managing-logs?hl=ja)

### 5.2. Cloud Monitoring APM によるサイトの信頼性のトラブルシューティング

Monitoring ワークスペースを作成して監視を行う。

- Cloud Console で、ナビゲーション メニュー > [モニタリング] をクリック
- ワークスペースがプロビジョニングされるまで待つ

次に、SLO と SLI を作成する。

- サービスレベル指標（SLI）
    - 例）リクエストのレイテンシ（リクエストに対してレスポンスを返すのにかかる時間）、エラー率、システム スループット（一般的には 1 秒あたりのリクエスト数）、可用性（サービスが使用可能な時間の割合）、耐久性（データが長期間にわたって保持される可能性）
- サービスレベル目標（SLO）
- サービスレベル契約（SLA）

例えば、SLO、SLI は以下のようなものである。

|SLI|指標|説明|SLO|
|---|---|---|---|
|リクエストのレイテンシ|フロントエンドのレイテンシ|ユーザーがページの読み込みを待機している時間。|過去 60 分間で 99% のリクエストが 3 秒以内に処理されている|
|エラー率|フロントエンドのエラー率|ユーザーが経験したエラー率。|過去 60 分間のエラー数が 0|
|エラー率|チェックアウトのエラー率|チェックアウト サービスを呼び出す他のサービスが経験したエラー率。|過去 60 分間のエラー数が 0|
|エラー率|通貨サービスのエラー率|通貨サービスを呼び出す他のサービスが経験したエラー率。|過去 60 分間のエラー数が 0|
|可用性|フロントエンドの成功率|サービスの可用性の判断基準として、成功したリクエストの割合。|過去 60 分間で 99% のリクエストが成功している|

Cloud Platform Console の Cloud モニタリング（ナビゲーション メニュー > [モニタリング]）で、左側のメニューにある [アラート]、[CREATE POLICY] の順にクリックし、SLO を監視するアラートを作成する。

## 6. ユースケース

### 6.1. Spinnaker と Kubernetes Engine を使用した継続的デリバリー パイプライン

まずは環境を設定する。

```bash
# ゾーンの設定
$ gcloud config set compute/zone us-central1-f

# プロジェクトを確認して環境変数へ
$ gcloud config list project
[core]
project = qwiklabs-gcp-04-4ceec88343aa
$ export PROJECT=qwiklabs-gcp-04-4ceec88343aa

# クラスタ作成
$ gcloud container clusters create spinnaker-tutorial \
    --machine-type=n1-standard-2
```

Spinnaker 用のサービスアカウントを作成して Cloud Storage にデータを保存できるようにする。

```bash
# Spinnaker 用のサービスアカウントの作成
$ gcloud iam service-accounts create spinnaker-account \
    --display-name spinnaker-account

# サービスアカウントのメールアドレスを環境変数に格納
$ export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')
# 取得できるメールアドレスはこんな「spinnaker-account@qwiklabs-gcp-00-c0fdab7fa347.iam.gserviceaccount.com」感じ

# サービスアカウントに Cloud Storage の権限を付与
$ gcloud projects add-iam-policy-binding $PROJECT \
    --role roles/storage.admin \
    --member serviceAccount:$SA_EMAIL

# サービスアカウントキーのダウンロード
$ gcloud iam service-accounts keys create spinnaker-sa.json \
     --iam-account $SA_EMAIL
```

Container Registry からの通知に使用する Cloud Pub/Sub トピックを作成する。

```bash
$ gcloud pubsub topics create projects/$PROJECT/topics/gcr
```

イメージの push についての通知を受け取れるように、Spinnaker から読み取ることができるサブスクリプションを作成する。

```bash
$ gcloud pubsub subscriptions create gcr-triggers \
    --topic projects/${PROJECT}/topics/gcr
```

Spinnaker のサービス アカウントに、gcr-triggers サブスクリプションからの読み取り権限を付与する。

```bash
$ export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')

$ gcloud beta pubsub subscriptions add-iam-policy-binding gcr-triggers \
    --role roles/pubsub.subscriber --member serviceAccount:$SA_EMAIL
```

Spinnaker をデプロイするために Helm を準備する。

```bash
# helm バイナリ取得・展開
$ wget https://get.helm.sh/helm-v3.1.1-linux-amd64.tar.gz
$ tar zxfv helm-v3.1.1-linux-amd64.tar.gz
$ cp linux-amd64/helm .

# Helm にクラスタの cluster-admin ロールを付与
$ kubectl create clusterrolebinding user-admin-binding \
    --clusterrole=cluster-admin --user=$(gcloud config get-value account)

# Spinnaker に cluster-admin ロールを付与し、すべての名前空間にリソースをデプロイできるようにする
$ kubectl create clusterrolebinding --clusterrole=cluster-admin \
    --serviceaccount=default:default spinnaker-admin

# Helm の使用可能なリポジトリに stable チャートのデプロイを追加（Spinnaker も含まれている）
$ ./helm repo add stable https://charts.helm.sh/stable
$ ./helm repo update
```

Spinnaker を作成する。

```bash
# 環境変数の準備
$ export PROJECT=$(gcloud info \
    --format='value(config.project)')
$ export BUCKET=$PROJECT-spinnaker-config

# Cloud Storage バケットの作成
$ gsutil mb -c regional -l us-central1 gs://$BUCKET
```

`spinnaker-config.yaml` ファイルの作成。

```bash
$ export SA_JSON=$(cat spinnaker-sa.json)
$ export PROJECT=$(gcloud info --format='value(config.project)')
$ export BUCKET=$PROJECT-spinnaker-config
$ cat > spinnaker-config.yaml <<EOF
gcs:
  enabled: true
  bucket: $BUCKET
  project: $PROJECT
  jsonKey: '$SA_JSON'

dockerRegistries:
- name: gcr
  address: https://gcr.io
  username: _json_key
  password: '$SA_JSON'
  email: 1234@5678.com

# デフォルトのストレージ バックエンドである minio を無効にします
minio:
  enabled: false

# GCP サービスを有効にするように Spinnaker を構成します
halyard:
  spinnakerVersion: 1.19.4
  image:
    repository: us-docker.pkg.dev/spinnaker-community/docker/halyard
    tag: 1.32.0
    pullSecrets: []
  additionalScripts:
    create: true
    data:
      enable_gcs_artifacts.sh: |-
        \$HAL_COMMAND config artifact gcs account add gcs-$PROJECT --json-path /opt/gcs/key.json
        \$HAL_COMMAND config artifact gcs enable
      enable_pubsub_triggers.sh: |-
        \$HAL_COMMAND config pubsub google enable
        \$HAL_COMMAND config pubsub google subscription add gcr-triggers \
          --subscription-name gcr-triggers \
          --json-path /opt/gcs/key.json \
          --project $PROJECT \
          --message-format GCR
EOF
```

Spinnaker チャートをデプロイする。

```bash
$ ./helm install -n default cd stable/spinnaker -f spinnaker-config.yaml \
           --version 2.0.0-rc9 --timeout 10m0s --wait
```

ポートフォワードして Spinnaker の管理画面を確認する。

```bash
$ export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \
    -o jsonpath="{.items[0].metadata.name}")
$ kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &
```

Cloud Shell ウィンドウの最上部で [ウェブでプレビュー] をクリックし、[ポート 8080 でプレビュー] をクリックすると Spinnaker の管理画面が見れる。  
次に Docker イメージを作成する。

```bash
# サンプルアプリの取得・展開
$ gsutil -m cp -r gs://spls/gsp114/sample-app.tar .
$ mkdir sample-app
$ tar xvf sample-app.tar -C ./sample-app
$ cd sample-app

# git リポジトリの作成
$ gcloud source repos create sample-app

# git 用アカウント情報設定・初回コミット・Push
$ git config --global user.email "$(gcloud config get-value core/account)"
$ git config --global user.name "[USERNAME]"
$ export PROJECT=$(gcloud info --format='value(config.project)')
$ git init
$ git add .
$ git commit -m "Initial commit"
$ git config credential.helper gcloud.sh
$ git remote add origin https://source.developers.google.com/p/$PROJECT/r/sample-app
$ git push origin master
```

次にビルドトリガーを構成する。  
Git タグが リポジトリに push されるたびに Docker イメージをビルドして push するように Container Builder を構成する。  
Container Builder はソースコードを自動的にチェックアウトして、リポジトリ内の Dockerfile から Docker イメージをビルドし、そのイメージを Google Cloud Container Registry に push する。

- Cloud Platform Console で、ナビゲーション メニュー > [Cloud Build] > [トリガー] をクリック
- [トリガーを作成] をクリック
- 次のトリガー設定を指定
    - 名前: sample-app-tags
    - イベント: 新しいタグを push する
    - 新しく作成した sample-app リポジトリを選択
    - タグ: v1.*
    - ビルド構成: Cloud Build 構成ファイル（yaml または json）
    - Cloud Build 構成ファイルの場所: /cloudbuild.yaml

これ以降、文字 "v" が先頭に付いている Git タグをソースコード リポジトリに push すると、Container Builder が自動的にアプリケーションをビルドして、Docker イメージとして Container Registry に push する。

Spinnaker で使用するために Kubernetes マニフェストを準備する。

```bash
$ export PROJECT=$(gcloud info --format='value(config.project)')
$ gsutil mb -l us-central1 gs://$PROJECT-kubernetes-manifests
$ gsutil versioning set on gs://$PROJECT-kubernetes-manifests
$ sed -i s/PROJECT/$PROJECT/g k8s/deployments/*
$ git commit -a -m "Set project ID"
```

Git タグを作成してイメージをビルドしてみる。

```bash
$ git tag v1.0.0
$ git push --tags
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 2 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (8/8), 778 bytes | 778.00 KiB/s, done.
Total 8 (delta 5), reused 0 (delta 0)
remote: Resolving deltas: 100% (5/5)
To https://source.developers.google.com/p/qwiklabs-gcp-00-c0fdab7fa347/r/sample-app
 * [new tag]         v1.0.0 -> v1.0.0
```

Spinnaker を管理する spin CLI をインストールする。

```bash
$ curl -LO https://storage.googleapis.com/spinnaker-artifacts/spin/1.14.0/linux/amd64/spin
$ chmod +x spin
```

spin を使用して、Spinnaker 内に sample という名前のアプリを作成。

```bash
$ ./spin application save --application-name sample \
                        --owner-email "$(gcloud config get-value core/account)" \
                        --cloud-providers kubernetes \
                        --gate-endpoint http://localhost:8080/gate
```

sample-app ソースコード ディレクトリから次のコマンドを実行します。これにより、サンプル パイプラインが Spinnaker インスタンスにアップロードされます。

```bash
$ export PROJECT=$(gcloud info --format='value(config.project)')
$ sed s/PROJECT/$PROJECT/g spinnaker/pipeline-deploy.json > pipeline.json
$ ./spin pipeline save --gate-endpoint http://localhost:8080/gate -f pipeline.json
```

パイプラインの実行を手動でトリガーして確認する。

- Spinnaker UI で、画面の上部にある [Applications] -> sampleをクリック。
- 上部にある [パイプライン] をクリックして、アプリケーションのパイプラインのステータスを表示。
- パイプラインを初めてトリガーするために、[Start Manual Execution] をクリック -> RUN 。
- [Execution Details] をクリックして、パイプラインの進行状況の詳細を表示。
- 進行状況バーで、デプロイメント パイプラインのステータスとそのステップを確認できる。
- ステージをクリックして、その詳細を表示します。
- 3～5 分後に統合テストフェーズが完了すると、パイプラインがデプロイメントの続行を手動で承認するよう求めてくる。
- 黄色の「人物」アイコンにカーソルを合わせ、[Continue] をクリック。
    - ロールアウトが本番環境フロントエンドと本番環境バックエンドのデプロイメントに進みます。これは数分後に完了します。
- Spinnaker UI の上部で [Infrastructure] > [Load Balancers] をクリックして、アプリを表示。
- ロードバランサのリストを下にスクロールし、[service sample-frontend-production] の下の [Default] をクリックするとロードバランサの詳細がページの右側に表示される。
- 右側の詳細ペインをスクロール ダウンし、Ingress IP のクリップボード ボタンをクリックしてアプリの IP アドレスをコピーする。
- コピーしたアドレスを新しいブラウザタブに貼り付けて、アプリケーションを表示
    - canary バージョンが表示される場合があるが、画面を更新すると production バージョンも表示される
- アプリケーションをビルド、テスト、デプロイするためのパイプラインを手動でトリガーした。

次に、コードの変更によるパイプラインのトリガーを行う。

```bash
# sample-app ディレクトリから次のコマンドを実行して、アプリの色をオレンジ色から青色に変更
$ sed -i 's/orange/blue/g' cmd/gke-info/common-service.go

# 変更にタグを付け、ソースコード リポジトリに push
$ git commit -a -m "Change color to blue"
$ git tag v1.0.1
$ git push --tags
```

- GCP Console で [Cloud Build] > [履歴] を開き、新しいビルドが表示されるまで数分待つ。（必要であればページを更新）  
- 新しいビルドが完了するまで待ってから次のステップに進む。
- Spinnaker UI に戻り、[Pipelines] をクリックして、イメージのデプロイを開始するパイプラインの動作を監視する。
    - 自動的にトリガーされたパイプラインは、表示されるまでに数分かかる。（必要であればページを更新）

次に、カナリア デプロイメントを観察する。

- デプロイが一時停止し、本番環境へのロールアウトを待機している間に、実行中のアプリケーションを表示しているウェブページに戻る。
    - アプリが表示されているタブを繰り返し更新。バックエンドのうちの 4 つがアプリの以前のバージョンを実行しており、カナリアを実行しているバックエンドは 1 つだけ。
    - 5 回更新するごとにアプリの新しい青色バージョンが表示されるはず。
- パイプラインが完了すると、アプリは以下のようになる。（コードを変更したため、色は青色。また、[バージョン] フィールドには canary ）
    - Backend that serviced this request
        - Pod Name: sample-backend-canary-xxxx
        - Node Name: gke-spinnaker-tutorial-default-pool-xxxx
        - Version: Canary
        - Zone: project/xxxx/zones/us-central1-f
        - Project: qwiklabs-gcp-00-xxxx
        - Node Internal IP: 10.128.0.4
        - Node External IP: 146.148.80.155
    - Frontend that handled this request
        - Frontend IP:PORT: 10.40.2.14.48884
        - Request to Backend: GET /metadata....
        - Error: None
- 必要に応じて前回の commit を元に戻し、この変更をロールバックできる。ロールバックすると新しいタグ　(v1.0.2）が追加され、v1.0.1 をデプロイするときに使用したのと同じパイプラインを通してタグが戻される。
    - `git revert v1.0.1`
    - Ctrl + O、ENTER、CTRL + X を押す
    - `git tag v1.0.2`
    - `git push --tags`
- ビルドとパイプラインが完了したら、ロールバックを確認する
    - [INFRASTRUCTURE] > [LOAD BALANCERS] の順にクリックしてから、[service sample-frontend-production] の [DEFAULT] をクリックして Ingress IP アドレスをコピー。このアドレスを新しいタブに貼り付け。
    - アプリがオレンジ色に戻り、production バージョン番号を確認できる。