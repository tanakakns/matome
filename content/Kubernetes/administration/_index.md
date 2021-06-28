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