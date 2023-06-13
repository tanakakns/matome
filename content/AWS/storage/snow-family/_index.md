---
title: "AWS Snowファミリー"
date: 2023-06-09T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

<!--more-->

[AWS Snowファミリー](https://aws.amazon.com/jp/snow/)についてまとめる。

- 1. [概要](#1-概要)

## 1. 概要

AWS Snowファミリーは、ネットワーク接続が難しい場合でも物理デバイスを使って大量のデータをAWSへ転送可能にするサービス。  
以下の種類がある。

- **AWS Snowcone**
  - Snowファミリーで最も小さく、8TBの使用可能なストレージ
- **AWS Snowball**
  - データ移行およびエッジコンピューティングデバイスの 2 つのオプション
  - アタッシュケースくらいのサイズ
  - **Snowball Edge Storage Optimized** ： 100 TB (80 TB 使用可能)
  - **Snowball Edge Compute Optimized** ： 42 TB (39.5 TB 使用可能)
- **AWS Snowmobile**
  - マルチペタバイトまたはEB（エクサバイト）規模のデジタルメディアの移行やデータセンターの停止に最適
  - ストレージとかアタッシュケースとかじゃなく、トラック