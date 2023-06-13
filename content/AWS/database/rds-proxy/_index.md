---
title: "Amazon RDS Proxy"
date: 2023-06-09T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

<!--more-->

[AWS RDS Proxy](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/rds-proxy.html)についてまとめる。

- 1. [概要](#1-概要)

## 1. 概要

**Amazon RDS Proxy** とは、Amazon RDS 向けの高可用性フルマネージド型データベースプロキシ。  
以下の機能を有する。

- コネクションプール
- 認証
  - データベースサーバーがパスワード認証しか対応していない場合でも、RDS Proxy への接続はIAM認証を利用することができる
- TLS/SSL が使える
- フェイルオーバーによる高可用性

サポートする RDS は以下の通り。

- RDS for MySQL 5.6 / 5.7
- Aurora MySQL バージョン1.x / 2.x
- RDS for PostgreSQL
- Aurora PostgreSQL 10.11以降 、11.5 以降

注意点は以下の通り。

- Aurora クラスタは、Writer エンドポイントのみ対応（Readerエンドポイントは未対応）
- Aurora サーバレスクラスタ、マルチマスタークラスタには未対応
- データベースと同じ VPC に存在する必要があり、パブリックアクセス不可