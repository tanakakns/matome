---
title: "クラスタ管理"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 6
---

<!--more-->

Application Lifecycle Management について。

## 1. Rolling Update and Rollbacks

Deployment のマニフェストの `spec.template.spec.containers.image` を変更し `kubectl apply` すれば、デフォルトの設定で **ローリングアップデート** される。  
もしくは `kubectl set image deployment/<deployment-name> <pod-name>=<image-name>` で直接イメージを変更しても同様。  
ロールアウトの確認方法は以下の通り。

```bash
# rollout の状況を確認
$ kubectl rollout status deployment <deployment-name>

# これまでの rollout を確認
$ kubectl rollout history deployment <deployment-name>

# ロールバック
$ kubectl rollout undo deployment <deployment-name>
```

なお、ローリングアップデートされる際は ReplicaSet が再度作成されている。

## 2. Cluster Maintainance

### 2.1. OS の更新

クラスタからノードが消失した場合、 `kube-scheduler` 起動時に設定したオプション `--pod-eviction-timeout=5m0s` が経過すると消失したノードに配置されていた Pod （ただし、 ReplicaSet の場合）は他のアクティブなノードで再作成される。（ ReplicaSet ではない場合は際作成されない）  
障害でノードが消失する場合は仕方ないが、メンテナンスでノードを停止する場合は、ノードから Pod を **ドレイン** して他のノードへ Pod を移す必要がある。

```bash
# 指定ノードの Pod を Graceful Shutdown させ、徐々に Pod を退避する
$ kubectl drain <node-name> [--ignore-daemonsets] [--force]
```

なお、ノードをクラスタに復帰させた場合は `uncordon` する。（ cordon は遮蔽 の意）

```bash
$ kubectl uncordon <node-name>

# ちなみに以下のコマンドはノードの Pod を直ちに停止させる（ Graceful Shutdown しない）
# そして、停止した Pod を他のノードに退避させることもない
$ kubectl cordon <node-name>
```

### 2.2. Kubernetes の更新

Kubernetes クラスタにインストール・実行されている各コンポーネントの更新方法について記載する。  
Kubernetes にインストールされているコンポーネントのバージョンは `kubectl get nodes` の `VERSION` 項目にて確認できる。  
ただし、 `etcd` や `CoreDNS` は `kube-apiserver` などの Kubernetes コンポーネントとは別プロジェクトであるためバージョニングが異なることに注意。

Kubernetes のコンポーネントは、マイナーバージョン 3 つまでであれば **ライブアップグレード** をサポートしている。  
しかしながら、一気にバージョンアップするのではなく、 1 つずつバージョンアップすることが推奨される。

- Master Nodes 、 Worker Nodes の順にアップグレードする
- Master Nodes 上で以下を実行する
    - OS を確認する `cat /etc/*release*/`
        - OS に応じたアップグレード手順を実施する必要があるため
        - 以下の参考 URL を見ながら実施すること
    - `kubeadm` を更新する： `apt-get upgrade -y kubeadm=1.12.0-00`
    - `kubeadm upgrade plan` でアップグレード情報を確認
    - `kubeadm upgrade apply v1.13.3` でアップグレードを実行
    - `kubectl get nodes` コマンド結果の `VERSION` を見てアップグレード出来ているか確認する
    - 必要であれば後述の `kubelet` `kubectl` のバージョンアップも実施する
- Worker Nodes を 1 つずつアップグレードする（バージョンアップしたノードを足して、古いノードを削除、を繰り返してもいい）
    - `kubectl drain <node-name>` @MasterNodes
    - `kubeadm` を更新する： `apt-get upgrade -y kubeadm=1.12.0-00`
    - `kubelet` を更新する： `apt-get upgrade -y kubelet=1.12.0-00`
    - 設定を更新する： `kubeadm upgrade node config --kubelet-version v1.12.0`
    - `kubelet` を再起動する： `systemctl restart kubelet`
    - `kubectl get nodes` コマンド結果の `VERSION` を見てアップグレード出来ているか確認する
    - `kubectl uncordon <node-name>` @MasterNodes

参考：[Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

## 3. バックアップとリストア

バックアップとリスト対象になるものは以下

- Resource Configuration
    - Github などで管理
    - 上記のように管理できていないものは `kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml` などで取得
- etcd Cluster
    - etcd の起動オプションに `--data-dir` があり、Master Nodes の各ディレクトリをバックアップする
    - `ETCDCTL_API=3 etcdctl snapshot save snapshot.db` コマンドでバックアップ可能
    - `ETCDCTL_API=3 etcdctl snapshot status snapshot.db` コマンドで内容を確認できる
    - `service kube-apiserver stop` コマンドで `kube-apiserver` を停止させた後、 `ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir=/var/lib/etcd-from-backup` でリストアする
    - etcd の起動オプション `--data-dir` や Static Pod であればマニフェストの設定を確認した後、 `systemctl daemon-reload && service etcd restart` で再起動する
    - 最後に `service kube-apiserver start` コマンドで `kube-apiserver` を起動
- Persistent Volumes

なお、  `etcdctl` コマンドを打ってもエラーが発生する場合は、下記のようにエンドポイントや証明書を指定すること。

```bash
$ ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/snapshot-pre-boot.db

$ ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot restore /opt/snapshot-pre-boot.db --data-dir=/var/lib/etcd-from-backup
```

参考： [Backing up an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)

## 4. セキュリティ

`kube-apiserver` に `kubectl` でアクセスする際は以下のセキュリティ事項がある。

- 誰がアクセスするのか -> Authentication / 認証
    - 以下のような方法がある
        - Username / Password
        - Username / Token
        - Certificates
        - ID Providor や LDAP
        - Service Accounts
- 何を実行するのか  -> Authorization / 認可・承認
    - 以下のような方法がある
        - RBAC Authorization ( Role Based Access Control)
        - ABAC Authorization ( Attribute Based Access Control)
        - Node Authorization
        - Webhook Mode

### 4.1. Authentication / 認証

**Account** には以下の 2 種類がある。

- **User**
    - 管理者、開発者
    - Kubernetes は User を管理しない
- **Service Accounts**
    - Bots など
    - Kubernetes は Service Accounts を管理する
        - `kubectl create serviceaccount sa1`
        - `kubectl get serviceacceount`

#### 4.1.1. パスワード・トークン認証

`kube-apiserver` が 静的ファイルを用いてユーザ認証する方法は以下の通り。  
まず、以下のファイルを準備する。

```csv:user-details.csv
# Password,UserName,UserID
password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
```

そして、上記のファイルを `kube-apiserver.service` の起動オプション `--basic-auth-file` に指定する。  
また、以下のように 4 項目目にグループを指定することもできるし、パスワードではなくトークンでもいい。（トークンファイルの場合は `--token-auth-file` オプションで指定 ）

```csv:user-token-details.csv
# Password,UserName,UserID,GroupID
token123,user1,u0001,group1
token123,user2,u0002,group1
token123,user3,u0003,group2
```

しかし、この方法は推奨されておらず、 Deprecated である。

#### 4.1.2. 証明書認証

まずは、 SSL/TSL 証明書まわりの復習。

- SSL/TLS Certificates まわりの登場人物
    - 秘密鍵： `openssl genrsa 1024 -out server.key` （ `.key` や `-key.pem` の形式が多い）
    - 公開鍵： `openssl rsa -in server.key -pubout > server.pem` （ `.pem` や `.crt` の形式が多い）
    - 証明書署名要求(CSR / Certificate Signing Request)： `openssl req -new -key server.key > server.csr` （ `.csr` の形式が多い）
    - 証明書： `openssl x509 -req -days 3650 -signkey server.key < server.csr > server.crt` （ `.crt` や `.pem` の形式が多い）
- 証明書の種類
    - サーバ証明書
        - サーバ側が提示する証明書、Webサイトの身分証
        - サーバ証明書は 認証局/CA が管理する
        - 暗号化に必要な公開鍵が含まれる
    - クライアント証明書
        - クライアント側が提示する証明書、クライアントの身分証
    - ルート証明書
        - 認証局/CA 自体の正当性をチェックする
        - サーバ証明書の検証のために使用する証明書
    - 自己証明証明書（オレオレ証明書）
        - 正式な認証局ベンダーでなく、個人で構築した認証局などで発行したルート証明書
- サーバ証明書
    - 「SSL/TSL による通信の暗号化」と「サーバ証明書による本人確認」を実施できる
    - 鍵の送受信は以下のステップ
        - ユーザは「▲公開鍵」「▼秘密鍵」「■サーバ証明書」を作成して、「▲公開鍵」「■サーバ証明書」をサーバに送付
        - サーバは「■サーバ証明書」を検証し、「●共通鍵」を生成して、「▲公開鍵」により暗号化した「●共通鍵」をユーザに送付
        - ユーザは受け取った「●共通鍵」を「▼秘密鍵」で復号化
        - 以降、データの送受信を行う際は「●共通鍵」で暗号化・復号化を行う

上記の証明書は以下のように利用される。

- `kube-apiserver` 、 `etcd` 、 `kubelet` はサーバ証明書を利用
- `kube-apiserver` にアクセスする admin ユーザ、 `kube-scheduler` 、 `kube-controller-manager` 、 `kube-proxy` 、 `kubelet` はクライアント証明書を利用
- `kubelet` 、 `etcd` にアクセスする `kube-apiserver` はクライアント証明書を利用

#### 4.1.3. 鍵・証明書の作成方法

上記の通り、サーバ・クライアント証明書が必要な訳だが、それらを管理する認証局/CA を Kubernetes クラスタに作成する必要がある。  
まずは、 CA の準備をする。

- CA の秘密鍵の作成： `openssl genrsa 2048 -out ca.key`
- CA の CSR の作成： `openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`
- CA の ルート証明書の作成： `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`
    - CA 自身が署名する自己証明書となる

なお、 `etcd` には別途 CA が必要であるため注意が必要。  
次に admin ユーザのクライアント証明書を作成する。

- admin ユーザの秘密鍵の作成： `openssl genrsa 2048 -out admin.key`
- admin ユーザの CSR の作成： `openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr`
    - `system:masters` の権限で `kube-admin` アカウントの要求になる
- admin ユーザのクライアント証明書の作成： `openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt`
    - CA に署名してもらう

上記の要領で他のクライアント証明書も作成する。  
なお、以上で作成した鍵・証明書を用いて kube-admin が `kube-apiserver` にアクセスする際は以下のようになる。

```bash
$ curl https://kube-apiserver:6443/api/v1/pods \
    --key admin.key \
    --cert admin.crt \
    --cacert ca.crt
```

```yaml:kube-config.yaml
# kubectl を使う場合の kube-config.yaml の設定例
apiVersion: v1
clusters:
- cluster:
    certificate-authority: ca.crt
    server: https://kube-apiserver:6443
  name: kubernetes
kind: Config
users:
- name: kubernetes-admin
  user: 
    client-certificate: admin.crt
    client-key: admin.key
```

次に `kube-apiserver` のサーバ証明書を作成する。  
（あまり知られていないが `kube-apiserver` のホスト名は `kubernetes`）  
（つまり、 `kubernetes` 、 `kubernetes.default` 、 `kubernetes.defalt.svc.cluster.local` のような感じで扱われる）

- `kube-apiserver` の秘密鍵の作成： `openssl genrsa 2048 -out apiserver.key`
- `kube-apiserver` の CSR の作成： `openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr`
- `kube-apiserver` の サーバ証明書の作成： `openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt`
    - CA に署名してもらう

なお、`kube-apiserver` でサーバ証明書を作成する際は `openssl.cnf` ファイルで以下のように設定しておく必要がある。（ `alt_names` に注目 ）

```txt:openssl.cnf
[req]
req_extensions = v3_req
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation,
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1   # GlobalIP
IP.2 = 172.17.0.87 # PriveteIP
```

以上のように作成した鍵・証明書を `apiserver.service` の起動オプションに付与する必要がある。  
他のサーバ証明書も同じ要領で作る。

#### 4.1.4. 鍵・証明書情報の確認方法

上記のような鍵・証明書を含めた Master/Worker Nodes の構築をスクラッチで行うのは所謂 **The Hard Way** 。  
ここでは、 `kubeadm` を用いて作成した場合の鍵・証明書情報の確認方法について見てみる。

- `etc/kubernetes/manifests/` 配下に Static Pod として作成されたコンポーネントのマニフェストがある
    - kube-apiserver.yaml 、など
    - 上記の yaml ファイルを確認すれば `spec.containers.command` に起動コマンドのオプションがあるので、そこで鍵・証明書のパスを確認できる
- `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -txt -noout` コマンドで中身が確認できる
- 以下に各コンポーネントの鍵・証明書の配置や発行者などを一覧化した Excel がある
    - https://github.com/mmumshad/kubernetes-the-hard-way/tree/master/tools

#### 4.1.5. Certificate API

jane というユーザが新規に admin ユーザになる場合を想定する。

- jane の秘密鍵を作成する： `openssl genrsa 2048 -out jane.key`
- jane の CSR を作成する： `openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr`
- jane の CertificateSigningRequest を作成する： `kubectl apply -f jane-csr.yaml`

```yaml:jane-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  signerName: kubernetes.io/kube-apiserver-client
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request: [ cat jane.csr | base64 の結果 ]
```

参考： [Kubernetes signers](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#kubernetes-signers)

- `kubectl get csr`
    - Pendding の jane を確認
- `kubectl certificate approve jane` ：承認
    - ちなみに `kubectl certificate deny jane` で却下できる
- `kubectl get csr jane -o yaml` ：クライアント証明書の取得
- クライアント証明書は Base64 になっているので `echo <cert_base64> | base64 --decode` でデコード

なお、上記の Certificate API は `kube-controller-manager` によって管理されているので、`kube-controller-manager` の起動オプション `--cluster-signing-cert-file` および `--cluster-signing-key-file` にそれぞれ CA のルート証明書と秘密鍵を設定しておく必要がある。

### 4.2. KubeConfig

デフォルトでは `$HOME/.kube/config` にある。  
内容は大きく `clusters` 、 `users` 、 `contexts` の 3 領域ある。

- `clusters` ：アクセス先のクラスタ情報
    - エンドポイントなどの情報はここ
- `users` ：アクセス先クラスタのユーザ情報
    - 鍵・証明書などの情報はここ
- `contexts` ： `clusters` と `users` の組合せ

```yaml:config
apiVersion: v1
kind: Config
current-context: ...

clusters:
- name: my-kube-playgroud # ▲
  cluster:
    certificate-authority: /path/to/ca.crt # certificate-authority-data: でベタ貼りもおk
    server: https://my-kube-playgroud:6443
users:
- name: my-kube-admin # ●
  user:
    client-certificate: /path/to/admin.crt
    client-key: /path/to/admin.key

- name: my-kube-admin@my-kube-playground
  context:
    cluster: my-kube-playgroud # ▲
    user: my-kube-admin        # ●
    namespace: ...
```

- `kubectl config view` コマンドで KubeConfig 情報を閲覧できる
- `kubectl config user-context <context>` で current-context の変更

### 4.3. RBAC

#### 4.3.1. Role と RoleBinding

以下のマニフェストで `Role` を作成できる。（ `kubectl apply -f developer-role.yaml` ）

```yaml: developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```

`Role` は `namespace` 毎に有効となる。  
次に上記で作成した `Role` を `User` に紐づけるための `RoleBinding` を作成する。（ `kubectl apply -f devuser-developer-binding.yaml` ）

```yaml:devuser-developer-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

以上で `Role` および `RoleBinding` は作成できる。  
なお、繰り返しになるが、 `Role` は `namespace` 毎に有効となるため、 `pods` や `services` などの `namespace` を指定して作成するオブジェクトが対象となる。  
これらは以下のコマンドで確認する。

```bash
# Role の確認
$ kubectl get roles
$ kubectl describe role developer

# RoleBinding の確認
$ kubectl get rolebindings
$ kubectl describe rolebinding devuser-developer-binding
```

なお、各ユーザは自分の権限を以下のコマンドで確認できる。

```bash
$ kubectl auth can-i create deployments
yes
 
$ kubectl auth can-i delete nodes
no
 
$ kubectl auth can-i create deployments --as dev-user
no
 
$ kubectl auth can-i create pods --as dev-user
yes
 
$ kubectl auth can-i create pods --as dev-user --namespace test
no
```

なお、 `kubectl api-resources` コマンドを利用するとリソースの一覧が取得できる。

```bash
$ kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
（省略）
```

#### 4.3.2. ClusterRole と ClusterRoleBinding

`namespace` 毎に作成するオブジェクトに対して指定する `Role` に対して、 `ClusterRole` はクラスタ横断（ `namespace` 横断）のオブジェクトに対して指定する。  
`nodes` や `PV` などである。

```yaml:cluster-admin-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

次に上記で作成した `ClusterRole` を `User` に紐づけるための `ClusterRoleBinding` を作成する。

```yaml:cluster-admin-role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

### 4.4. Image Security

普段マニフェストに記載している `image: nginx` は省略系で、フルで記載すると `image: docker.io/nginx/nginx` となる。（なお、 `docker.io` は Docker Hub の URL）  
フォーマットとは `image: <Registry>/<User Account>/<Image Repository>` で `nginx` は `<Image Repository>` のみ記載している。  
Private Registory を利用する場合、認証が必要になるが、その際はどうするのか？  
以下の通り `Secret` を利用することになる。

```bash
# Private Registory 認証用の Secret 作成
$ kubectl create secret docker-registry regcred \
    --docker-server=private-registry.io \
    --docker-username=registry-user \
    --docker-password=registry-password \
    --docker-email=registry-user@org.com
```

上記の Secret を Pod のマニフェストの `spec.imagePullSecrets` に指定する。

### 4.5. Security Context

Pod マニフェストの `securityContext` 項目を設定することにより、コンテナで実行されるプロセスの実行ユーザなどを指定することができる。

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  fsGroupChangePolicy: "OnRootMismatch"
```

なお、ユーザを指定しない場合は `root` ユーザとなる。

### 4.6. Network Policy

NetworkPolicy を適用すれば、 Pod に対して Inress / Egress のトラフィックを制限することができる。  
なお、 NetworkPolicy を適用しない場合、全てのトラフィックが許可される。

- Ingress
    - 自分から見て、リクエストを送信するトラフィック
    - Egress に対するレスポンスは Ingress でないことに注意
- Egress
    - 自分から見て、リクエストを送信するトラフィック
    - Ingress に対するレスポンスは Egress でないことに注意

例えば、以下の NetworkPolicy は Port 3306 の DB サーバに対して、 api-server からのトラフィックを許可し、その他は遮断するポリシー。  
（バックアップ用に DB から cidr:192.168.5.10/32 帯にある Port 80 にアクセスできるようにしている。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:     # policy を適用する Pod の選択
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector: # Ingress を許可する Pod の選択
        matchLabels:
          name: api-pod
      namespaceSelector: # 「-」を見れば分かるがこのルールは上記の PodSelector と and 条件
        matchLabels:
          name: prod
    - ipBlock: # これは or 条件
        cidr: 192.168.5.10/32
    ports:          # Ingress を許可する Port
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```

ネットワークソリューションによっては NetworkPolicy をサポートしていないものがあるので注意。（例えば、 Flannel ）

```bash
$ kubectl get networkpolicies
```

## 5. Storage

Docker の場合、以下のコマンドを実行することによってコンテナ内にホストのディレクトリをマウントできる。

```
$ docker run mysql -v data_volume:/var/lib/mysql
```

- ホストコンピュータの Docker インストールディレクトリに `volumes` というディレクトリがある
- `volumes` ディレクトリ内に `data_volume` というディレクトリが作成され、コンテナに関係なく永続化される
- ホストの `data_volume` ディレクトリが、 コンテナの `/var/lib/mysql` ディレクトリにマウントされる

といった動きになる。

```
$ docker run mysql -v /data/mysql:/var/lib/mysql
```

- 上記のように実行した場合は、ホストの `/data/mysql` ディレクトリが、コンテナの `/var/lib/mysql` ディレクトリにマウントされる

つまり、 **ホストのとあるディレクトリを Volume と定義して、それをコンテナにマウントする** のが Volume の仕組みだ。  
厳密にいうと、前者の例（ `data_volume` ）のように Volume を定義してマウントする方法を **ボリュームマウンティング** 、後者の例（ `/data/mysql` ）のようにディレクトリを直接マウントする方法を **バインドマウンティング** という。  
厳密なコマンドに直すと以下のようになる。

```bash
# バインドマウンティング
$ docker run mysql --mount type=bind,source=/data/mysql,target=/var/lib/mysql
```

以上のような Docker の volume 操作は Storage Drivers で実現される。  
Storage Driver は Docker プロセスがイメージやコンテナに加えられた変更を差分形式で保持する方式で、上記で 「ディレクトリをマウント」と表現してきたが厳密にはそうではない。  
一方 Volume Drivers というものがあり、これは Docker プロセスを経由せずとも、直接コンテナからホストのディレクトリをマウントできる仕組みだ。

- Storage Drivers
    - AUFS, ZFS, BTRFS, Device Mapper, Overlay, Overlay2 など
- Volume Drivers ( Volume Plugin )
    - Local, Azure File Storage, Convoy, DigitalOcean Block Storage, Flocker, gce-docker, GlusterFS, NetApp, RexRay, Portworx, VMware vSphere Storage など

Docker で Volume Drivers を利用する方法は以下のようになる。

```
$ docker run mysql -it \
    --name mysql \
    --volume-driver rexray/ebs |
    --mount src=ebs-vol,target=/var/lib/mysql
```

Kubernetes からは CSI (Container Storage Interface) 準拠した形で上記の機能が呼び出されることになる。

### 5.1. Volumes

Volume は Pod の `spec.volumes` で定義し、 `spec.containers.volumeMouts` で Pod にマウントする。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```

上記の `type: Directory` の Volume の場合、 Pod がスケジューリングされた Node の `/data` をマウントする訳だが、同じ ReplicaSet の Pod が同じ Node へスケジューリングされる補償はない。  
例えば、下記のように AWS S3 を Volume として定義して利用することもできる。

```yaml
volumes:
- name: data-volume
  awsElasticBlockStore:
  hostPath:
    volumeID: <volume-id> path: /data
    fsType: ext4
```

### 5.2. Persistent Volumes

先の Volume では、 Pod マニフェストの `spec.volumes` に定義するため、 Pod 毎にいちいち設定しなければならない。  
Volume の一元管理が可能となるオブジェクトとして **PersistentVolume** がある。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

```bash
$ kubectl create –f pv-definition.yaml
$ kubectl get persistentvolume
```

### 5.3. Persistent Volume Claims

Claim は「請求」の意。  
**PersistentVolumeClaim** を利用すれば、 Kubernetes 上に作成された PersistentVolume の中から条件に合うものを「請求」して取得することができる。

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

例えば、ある PVC を apply して様子を見てみる。

```bash
$ kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/pv0001   1Gi        RWO            Delete           Bound    default/pv-slow-claim   slow                    6m24s

NAME                                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pv-slow-claim   Bound    pv0001   1Gi        RWO            slow           42s
```

PVC の条件に合う PV である pv0001 が選ばれ、 PVC の VOLUME 項目に pv0001 が割り当てられているのが分かる。  
この PVC を利用して Pod から PV を「請求」するのは以下の通り。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-slow-test
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["tail", "-f", "/dev/null"]
    volumeMounts:
    - name: claim-volume
      mountPath: /data
  volumes:
  - name: claim-volume
    persistentVolumeClaim: # ここで「請求」してる
      claimName: pv-slow-claim
```

マニフェストだけ見ても PVC だけなので、どの PV が割り当てられるか分からない。  
上記のマニフェストを apply した後、 describe コマンドなどで確認するしかない。  
つまり、 **PV は静的** なのに対して、 **PVC は動的** な Volume 取得方法となる。

### 5.4. Storage Class

PV は静的なオブジェクトなので、例えば、 GCP などのクラウドのストレージをプロビジョンする必要がある場合はその実行を別途行わなければならない。  
**StorageClass** はクラウドなどプロビジョニングを要するストレージを動的に確保してくれる。

```yaml:sc-definition.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd # これを指定する必要あり
```

上記の StorageClass を利用して、 PVC を作成する場合は `storageClassName` 項目で作成した StorageClass を指定する。

```yaml:pvc-definition.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  storageClassNme: google-storage # これ
  resources:
    requests:
      storage: 500Mi
```

## 6. Networking

Linux のネットワーク系基本コマンドは以下の通り。

- `ip link` , `ifconfig -a` , `cat /etc/network/interfaces`
- `ip addr`
- `ip addr add 192.168.1.10/24 dev eth0`
- `ip route` : default gateway
- `route`
- `ip route add 192.168.1.0/24 via 192.168.2.1`
- `cat /proc/sys/net/ipv4/ip_forward`

Linux の DNS 系基本コマンドは以下の通り。

- `cat "192.168.1.11 db" >> /etc/hosts`
- `cat "nameserver 192.168.1.100" >> /etc/resolv.conf`
- `nslookup www.google.com`
- `dig www.google.com`

また、DNS レコードの `A` とか `AAAA` とか `CNAME` とか復習せんとあかんかも、、、

### 6.1. Pod Networking

CNI は `kubelet` の以下のオプションによって設定される。

```bash
$ kubelet \
    --network-;lugin=cni \
    --cni-conf-dir=/etc/cni/net.d \
    --cni-bin-dir=/etc/cni/bin # /opt/cti/bin
$ ./net-script.sh add <container> <namespace>
```

CNI 設定ファイルは JSON 形式で以下のようになっている。

```json
{
   "cniVersion":"0.2.0",
   "name":"mynet",
   "type":"bridge",
   "bridge":"cni0",
   "isGateway":true,
   "ipMasq":true,
   "ipam":{
      "type":"host-local",
      "subnet":"10.22.0.0/16",
      "routes":[
         {
            "dst":"0.0.0.0/0"
         }
      ]
   }
```

### 6.2. weave

weave は CNI Plugin の 1 つ。  
ここでは weave plugin の設定についてみる。  
weave のエージェントは Demonset として 各ノートに deploy する。

```bash
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
$ kubectl get pods –n kube-system
```

### 6.3. Service Networking

`kube-proxy` には以下のモードがあり、 default は iptables 。

```bash
$ kube-proxy --proxy-mode [userspace | iptables | ipvs ] ...
```

### 6.4. DNS

web-service という Service があった場合、以下のように名前登録される。

- web-service
    - ただし、これは default namespace のみ
- web-service.<namespace>
- web-service.<namespace>.svc
- web-service.<namespace>.svc.cluster.local

IP アドレスが 10.244.2.5 の Pod があった場合、以下のように名前登録される。

- 10-244-2-5
    - ただし、これは default namespace のみ
- 10-244-2-5.<namespace>
- 10-244-2-5.<namespace>.pod
- 10-244-2-5.<namespace>.pod.cluster.local

Kubernetes の代表的な DNS である CoreDNS は 2 冗長化された Deployment と ClusterIP として kube-system namespace にデプロイされる。  
`/etc/coredns/Corefile` あたりに設定ファイルがある。もしくは coredns という ConfigMap がある。

### 6.4. Ingress

Kubernetes のサービスを インターネットに公開する場合、 Service の LoadBalancer を利用する。  
LoadBalancer は内部的には NodePortで構成され、 GCP などのプロバイダがロードバランサを構成し、各ノードの NodePort にトラフィックを転送してくれることで成り立っている。（設定だけみると、ただの Service の LoadBalancer だけみ見えるが、じつは、）  
しかし、以下の場合はどうだろう。

- サービスの URL：「https://my-online-store.com」
- 「https://my-online-store.com/apparel」に該当する Service(LoadBalancer) がある
- 「https://my-online-store.com/video」に該当する Service(LoadBalancer) がある

上記の場合、どのように振り分けを行うのか。  
こういう場合に **Ingress** を用いる。  
内部的には、Nginx などで実現した **Ingress Controller** と、 ルーティングルールの設定を記載する **Ingress Resource** からなる。  
Nginx の場合の Ingress Controller は以下のような Deployment / ConfigMap / Service(NodePort) / ServiceAccont で実現することになる。

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spac:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
      - name: nginx-ingress-controller
        image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
        args:
        - /nginx-ingress-controller
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              filePath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              filePath: metadata.namespace
        ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configuration
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NordPort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selectors:
    name: nginx-ingress
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
```

Ingress Controller 用の Image を使うなど決まりがある。  
Ingress Resource は以下のように作成する。

```yaml:Ingress-wear.yaml
# 振り分けが無い場合
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    servicePort: 80
```

```yaml:Ingress-wear-watch.yaml
# パスベースの振り分けの場合
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          servicePort: 80
```

```yaml:Ingress-wear-watch.yaml
# ホスト名ベースの振り分けの場合
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80
```

```bash
$ kubectl apply -f Ingress-wear.yaml
$ kubectl get ingress # Ingress Resource のことが ingress
```

上記のように、振り分けルールに応じて作成する。

## 7. kubeadm

`kubeadm` を使って Kubernetes クラスタを構築してみる。  
以下に 1 Master/2 Worker を構築する Vagrant がある。

- https://github.com/mmumshad/certified-kubernetes-administrator-course