---
title: "アプリケーション管理"
date: 2021-01-30T18:52:48+09:00
draft: false
hide:
- toc
- nextpage
weight: 6
---

<!--more-->

1. リリース
2. スケールイン・アウト

## 1. リリース

### 1.1. Rolling Update and Rollbacks

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

## 2. スケールイン・アウト

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