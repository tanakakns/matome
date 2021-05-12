---
title: "Memorystore"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 3
---

[Memorystore](https://cloud.google.com/memorystore/docs)

<!--more-->

Memorystore には [Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis) と [Memorystore for Memcached](https://cloud.google.com/memorystore/docs/memcached) の 2 種類ある。  
ここでは、前者をベースに記載する。

1. コンセプト

## 1. コンセプト

- 自作の VPC ではなく、 Google 管理の VPC や Google サービスネットワーク に作成される
    - **リージョン** と **ゾーン** （代替ゾーン） を指定して作成できる
    - VM インスタンスから接続するには 同一リージョン に作成する必要がある
- [接続モード](https://cloud.google.com/memorystore/docs/redis/networking?hl=ja#connection_modes)
    - **ダイレクトピアリング**
        - 自身の VPC と Google 管理の VPC 間で **VPC ピアリング** を作成して接続する
    - **プライベートサービスアクセス**
        - VPC の [プライベートサービスアクセス](https://cloud.google.com/vpc/docs/private-services-access?hl=ja) で接続する
        - こちらを利用すると Cloud VPN や Cloud Interconnect 経由でも接続できる
        - 共有 VPC になら VPC 内に Memorystore を作成できる？？？
- 注意点
    - App Engine には Memorystore とは別途 [App Engine memcache](https://cloud.google.com/appengine/docs/standard/php/memcache) というサービスがある
        - 現状は推奨されておらず、 Memorystore を利用することが推奨されている
        - App Engine memcache にはサービスレベル別に **共有 memcache** 、 **専用 memcache** があり、名前の通り後者の方がサービスレベルが高い