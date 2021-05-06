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

Compute Engine、Kubernetes Engine、Cloud Run、App Engine、Cloud Functions

<!--more-->

|サービス|モデル|形式|ユースケース|対応AWS|
|:---|:---|:---|:---|:---|
|[Compute Engine](./compute-engine)|IaaS|VMイメージ|一般的なワークロード|EC2|
|[Kubernetes Engine](./kubernetes-engine)|IaaS/PaaS|クラスタ|コンテナワークロード|EKS/ECS|
|Cloud Run|PaaS|コンテナ|コンテナワークロード|Fargate|
|[App Engine](./app-engine) フレキシブル環境|PaaS|アプリ/コンテナ|HTTPサービス/バックエンドアプリ|Elastic Beanstalk|
|[App Engine](./app-engine) スタンダード環境|PaaS|アプリ|HTTPサービス/バックエンドアプリ|Elastic Beanstalk|
|[Cloud Functions](./cloud-functions)|FaaS|関数|イベントドリブンな操作|Lambda|