---
title: "コマンド"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 4
---

`kubectl` について。

<!--more-->

[kubectl リファレンス](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
[kubectlチートシート](http://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/)
[Kubernetes道場 23日目 - kubectlを網羅する](https://cstoku.dev/posts/2018/k8sdojo-23/)

## 全体像

```zsh
% kubectl --help
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create        ファイルまたは標準入力からリソースを作成する
  expose        ReplicaSet、Service、Deployment、Pod に対して新しい Service を作成する
  run           イメージをクラスタ上に起動する
  set           オブジェクトに特定の features を設定する

Basic Commands (Intermediate):
  explain       リソースの説明を表示する
  get           1つまたは複数のリソースを表示する
  edit          サーバ上のリソースを編集する
  delete        ファイル名、標準入力、リソース、名前、セレクタをもとにリソースを削除する

Deploy Commands:
  rollout       リソースのロールアウトを管理する
  scale         Deployment、 ReplicaSet のサイズを設定する
  autoscale     Deployment、 ReplicaSet をオートスケールする

Cluster Management Commands:
  certificate   Modify certificate resources.
  cluster-info  クラスターの情報を表示する
  top           Display Resource (CPU/Memory/Storage) usage.
  cordon        Mark node as unschedulable
  uncordon      Mark node as schedulable
  drain         Drain node in preparation for maintenance
  taint         Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe      Show details of a specific resource or group of resources
  logs          Print the logs for a container in a pod
  attach        Attach to a running container
  exec          Execute a command in a container
  port-forward  Forward one or more local ports to a pod
  proxy         Run a proxy to the Kubernetes API server
  cp            Copy files and directories to and from containers.
  auth          Inspect authorization
  debug         Create debugging sessions for troubleshooting workloads and nodes

Advanced Commands:
  diff          Diff live version against would-be applied version
  apply         ファイル名または標準入力でリソースにコンフィグを適用する
  patch         Update field(s) of a resource
  replace       Replace a resource by filename or stdin
  wait          Experimental: Wait for a specific condition on one or many resources.
  kustomize     Build a kustomization target from a directory or a remote url.

Settings Commands:
  label         リソースのラベルを更新する
  annotate      リソースのアノテーションを更新する
  completion    Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  api-resources Print the supported API resources on the server
  api-versions  Print the supported API versions on the server, in the form of "group/version"
  config        kubeconfigを変更する
  plugin        Provides utilities for interacting with plugins.
  version       Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

## 基本設定

`kubectl` コマンドの設定情報は `${HOME}/.kube/config` にある。  
これを `kubeconfig` と呼ぶ。

```zsh
current-context current-contextを表示する
delete-cluster  指定したコンテキストをkubeconfigから削除する
delete-context  指定したコンテキストをkubeconfigから削除する
delete-user     Delete the specified user from the kubeconfig
get-clusters    kubeconfigで定義されたクラスターを表示する
get-contexts    1つまたは複数のコンテキストを記述する
get-users       Display users defined in the kubeconfig
rename-context  Renames a context from the kubeconfig file.
set             kubeconfigに個別の変数を設定する
set-cluster     kubeconfigにクラスターエントリを設定する
set-context     kubeconfigにコンテキストエントリを設定する
set-credentials kubeconfigにユーザーエントリを設定する
unset           kubeconfigから変数を個別に削除する
use-context     kubeconfigにcurrent-contextを設定する
view            マージされたkubeconfigの設定または指定されたkubeconfigを表示する
```

kind 環境でやる。

```zsh
% kubectl config current-context
kind-kind

% kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:50584
KubeDNS is running at https://127.0.0.1:50584/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### context

`kubecnfig` に含まれる設定は `contexts` `clusters` `users` の 3 種類で、 `contexts` に `cluster` （接続先クラスタ情報）と `users` （ credential 情報）が含まれる。

基本設定でも特に `context` 設定については以下だけ覚えておく。

```zsh
% kubectl config view            # kubeconfig の表示

% kubectl config current-context # 現在の context を取得
% kubectl config get-contexts    # context の一覧を取得
% kubectl config set-context     # context をエントリ
% kubectl config use-context     # 現在の context に設定
```

なお、以降のコマンドで `-o yaml --dry-run=client` をつけるとマニフェストを確認できるので試してみるといい。

## `kubectl create`

Pod 以外の大体のリソースは `kubectl create` で作れる。[参考](https://kubernetes.io/docs/reference/kubectl/conventions/#generators) 

- clusterrole: Create a ClusterRole.
- clusterrolebinding: Create a ClusterRoleBinding for a particular ClusterRole.
- configmap: Create a ConfigMap from a local file, directory or literal value.
- cronjob: Create a CronJob with the specified name.
- deployment: Create a Deployment with the specified name.
- job: Create a Job with the specified name.
- namespace: Create a Namespace with the specified name.
- poddisruptionbudget: Create a PodDisruptionBudget with the specified name.
- priorityclass: Create a PriorityClass with the specified name.
- quota: Create a Quota with the specified name.
- role: Create a Role with single rule.
- rolebinding: Create a RoleBinding for a particular Role or ClusterRole.
- secret: Create a Secret using specified subcommand.
- service: Create a Service using specified subcommand.
- serviceaccount: Create a ServiceAccount with the specified name.

```zsh
# Deployment
% kubectl create deployment nginx --image=nginx --replicas=1

# Job
% kubectl create job busybox --image=busybox -- /bin/sh -c 'date; echo Hello'

# CronJob
% kubectl create cronjob busybox --image=busybox --schedule="*/10 * * * *"  -- /bin/sh -c 'date; echo Hello'

# ConfigMap
% kubectl create configmap my-config --from-literal=key1=config1 --from-literal=key2=config2

# Secret
% kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret
```

なお、 Pod だけを作りたい場合は `kubectl run` を利用する。

## `kubectl run`

`kubectl create` がだいたいなんでも作れたのに対し、 `kubectl run` は Pod のみの実行に特化。  
オプションで Service をつけれたりもするが、基本は `kubectl create` でリソースを作成するのがよい。

```zsh
# Pod
% kubectl run nginx --image=nginx

# PodとServiceを同時に作成
% kubectl run nginx --image=nginx --port=80 --expose
```

## `kubectl apply`

マニフェストを適用するコマンド。  
`kubectl create` でもマニフェスト適用できるが、 `kubectl apply` を利用すること。  
理由は以下の通り。

- `kubectl apply` は差分を適用するため、差分が無い場合は何もしない
- `kubectl apply` は「前回適用したマニフェスト」「現在クラスタに登録されているリソースの状態」「今回適用するマニフェスト」の 3 種類から差分が算出される
- `kubectl create` は「前回適用したマニフェスト」として保存されない

## `kubectl label`

リソースに対して label の追加・更新・削除を行う。

## `kubectl set`

オブジェクトに特定の features を設定するコマンド。  
例えば、以下のコマンドはデプロイメントのイメージのみ変更する。

```bash
$ kubectl set image deployment/<demployment> <new-image>
```