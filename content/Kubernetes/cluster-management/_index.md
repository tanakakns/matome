---
title: "クラスタ管理"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 7
---

<!--more-->

1. ノードの更新
2. コンポーネントの更新
3. バックアップとリストア
4. Storage
5. Networking
6. kubeadm
7. Troubleshooting
8. Advanced kubectl

## 1. ノードの更新

クラスタからノードが消失した場合、 `kube-scheduler` 起動時に設定したオプション `--pod-eviction-timeout=5m0s` が経過すると消失したノードに配置されていた Pod （ただし、 ReplicaSet の場合）は他のアクティブなノードで再作成される。（ ReplicaSet ではない場合は際作成されない）  
障害でノードが消失する場合は仕方ないが、 OS 更新などのメンテナンス作業でノードを停止する場合は、ノードから Pod を **ドレイン** して他のノードへ Pod を移す必要がある。

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

## 2. コンポーネントの更新

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

## 4. Storage

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

### 4.1. Volumes

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

### 4.2. Persistent Volumes

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

### 4.3. Persistent Volume Claims

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

### 4.4. Storage Class

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

## 5. Networking

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

### 5.1. Pod Networking

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

### 5.2. weave

weave は CNI Plugin の 1 つ。  
ここでは weave plugin の設定についてみる。  
weave のエージェントは Demonset として 各ノートに deploy する。

```bash
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
$ kubectl get pods –n kube-system
```

### 5.3. Service Networking

`kube-proxy` には以下のモードがあり、 default は iptables 。

```bash
$ kube-proxy --proxy-mode [userspace | iptables | ipvs ] ...
```

### 5.4. DNS

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

### 5.4. Ingress

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

## 6. kubeadm

`kubeadm` を使って Kubernetes クラスタを構築してみる。  
以下に 1 Master/2 Worker を構築する Vagrant がある。

- https://github.com/mmumshad/certified-kubernetes-administrator-course

## 7. Troubleshooting

### 7.1. Application Failure

- `kubectl get all` で全体感を確認
- `name` や `port` に誤りが無いか確認
- Service の Selector に矛盾がないか確認
- DB の ID/Pass などといった環境変数に誤りが無いか確認

### 7.2. Control Plane Failure

- `kubectl get po -n kube-system` で全コンポーネントの稼働を確認
- 各のプロセスの状態を確認
    - `service kube-apiserver status`
    - `service kube-controller-manager status`
    - `service kube-scheduler status`
- Worker Nodes のコンポーネントについても忘れず確認
    - `service kubelet status` / `systemctl status kubelet`
    - `service kube-proxy status`
- さらにログを確認
    - `kubectl logs kube-apiserver-master -n kube-system`
    - `sudo journalctl -u kube-apiserver`
        - **systemd** で管理されたデーモンのログを見るコマンド
        - `/usr/lib/systemd/system/xxxx.service` という形式で **unit** という単位で管理される（だから `-u` ）
            - `etc/systemd/system/xxxx.service.d`
        - unit の管理には `systemctl` コマンドを利用する

serviceやsystemctlとは何か？  
serviceとsystemctlは、どちらも Linux で楽にコマンドを実行するためのプログラム。  

- service
    - /etc/init.dにあるシェルスクリプトのファイルを指定することで、シェルを実行してくれるもの
    - 例えば、MySQLの実行時にservice mysql startとすると思うが、これは/etc/init.d/mysqlと言うシェルスクリプトにstartと言う引数を与えることで、mysqlを実行している
- systemctl
    - /lib/systemdにある設定ファイルを指定して、コマンドを実行するもの
    - serviceと大きく異なる点は、serviceはシェルを実行するのに対してsystemctlは独自の設定ファイルを使って実行する点だ

どちらも使える状態であれば、systemctlを使うのが望ましい。

### 7.3. Woker Nodes Failure

- `kubectl get nodes` でノードの状態を確認
- `Ready` では無いノードに対して `kubectl describe node <node-name>` して `Unknown` の項目を確認
- ノードで `top` `df -h` などしてリソースの状態を確認
- `kubelet` の状態を確認
    - `service kubelet status` / `systemctl status kubelet`
    - `sudo journalctl -u kubelet`
        - Shift + G でボトムが見れる
- `kubelet` の証明書を確認
    - `openssl x509 -in /var/lib/kubelet/woker-1.crt -text`
- `kubectl cluster-info` でクラスタに関する設定が見れる

### 7.4. Network Troubleshooting

- CNI
    - `kubelet` のオプション `--cni-bin-dir` や `--network-plugin` を確認
    - CNI 系の Pod が稼働していることも確認
- DNS
    - kubeDNS や coreDNS が稼働していることを確認（ Deployments ）
    - 各々の設定ファイルや ConfigMap を確認
    - `proxy . /etc/resolve.conf`
- `kube-proxy`
    - DaemonSet になっているので確認

## 8. Advanced kubectl

### 8.1. JSON PAH

**JSON PATH** を利用したコマンド。（ JSON PATH は `metadata.name` みたいなやつ）  
`kubectl` は `kube-apiserver` と JSON 形式で通信しているが、コマンドの結果はユーザに見やすい表形式で、且つ、間引いた情報で出力されている。  
`-o wide` オプションである程度詳細情報まで表示できるが、 `-o json` オプションを利用すれば全情報が JSON 形式で出力される。  
しかし、あまりにも情報が多いので `-o=jsonpath=` オプションと JSON PATH を利用して情報をフィルタリングできる。

```bash
# シングルクオート と 中括弧 で囲む
$ kubectl get po -o=jsonpath='{ .items[0].spec.containers[0].image }'
# 中括弧単位で複数の JSON PATH を指定できる
$ kubectl get po -o=jsonpath='{ .items[*].metadata.name }{ .items[*].status.capacity.cpu }'
# 改行を挟んで見やすくできる
$ kubectl get po -o=jsonpath='{ .items[*].metadata.name }{"\n"}{ .items[*].status.capacity.cpu }'
```

`range` などの JSON PATH の演算子も利用できる。  
他にも `-o=custom-columns=` 、 `--sort-by=.metadata.name` などのオプションもある。

### 8.2. check kubeconfig

```bash
$ kubectl cluster-info --kubeconfig /root/admin.kubeconfig
```

## 9. etc

- [「Kubernetes the hard way」を図で理解したい -前編-](https://nishipy.com/archives/1233)
- [「Kubernetes the hard way」を図で理解したい -後編-](https://nishipy.com/archives/1304)