---
title: "Amazon API Gateway"
date: 2023-06-13T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 5
---

<!--more-->

[Amazon API Gateway](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/welcome.html)についてまとめる。

- 1. [概要](#1-概要)
- 2. [APIエンドポイントタイプ](#2-APIエンドポイントタイプ)

## 1. 概要

**Amazon API Gateway** は、あらゆる規模の REST、HTTP、および WebSocket API を作成、公開、維持、モニタリング、およびセキュア化するための AWS のサービス。

## 2. APIエンドポイントタイプ

- **エッジ最適化** API エンドポイント（デフォルト）
  - 地理的に分散されたクライアントに最適
  - 最寄りの CloudFront POP (Point Of Presence) にルーティングされる
- **リージョン** API エンドポイント
  - 同じリージョンのクライアントを対象
- **プライベート** API エンドポイント
  - VPC内 からしかアクセスできない API エンドポイント