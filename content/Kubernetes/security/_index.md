---
title: "セキュリティ"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 8
---

<!--more-->

1. Authentication / 認証

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

しかし、この方法は推奨されておらず、 **Deprecated** である。

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