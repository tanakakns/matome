---
title: "コンピューティングとホスティング"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

1. [サービス一覧](#1-サービス一覧)

{{% children description="true"   %}}

<!--more-->

## 1. サービス一覧

[Google Cloud のコンピューティング プロダクト](https://cloud.google.com/products/compute)

|サービス|モデル|形式|ユースケース|対応AWS|
|:---|:---|:---|:---|:---|
|Compute Engine / GCE|IaaS|VMイメージ|一般的なワークロード|EC2|
|Kubernetes Engine / GKE|IaaS/PaaS|クラスタ|コンテナワークロード|EKS/ECS|
|Cloud Run|PaaS|コンテナ|コンテナワークロード|Fargate|
|App Engine フレキシブル環境|PaaS|アプリ/コンテナ|HTTPサービス/バックエンドアプリ|Elastic Beanstalk|
|App Engine スタンダード環境|PaaS|アプリ|HTTPサービス/バックエンドアプリ|Elastic Beanstalk|
|Cloud Functions|FaaS|関数|イベントドリブンな操作|Lambda|



