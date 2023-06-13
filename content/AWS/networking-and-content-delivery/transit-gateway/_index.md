---
title: "AWS Transit Gateway"
date: 2023-06-09T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

<!--more-->

[AWS Transit Gateway](https://docs.aws.amazon.com/ja_jp/vpc/latest/tgw/what-is-transit-gateway.html)についてまとめる。

- 1. [概要](#1-概要)

## 1. 概要

**Transit Gateway** は、VPC とオンプレミスネットワークを相互接続するために使用できるネットワークの中継ハブ。

- **アタッチメント** — 以下をアタッチできる
  - VPC 接続
  - VPN 接続
  - Direct Connectゲートウェイ
  - 別のTransit Gateway とのピア接続
  - 接続 SD-WAN/サードパーティー製ネットワークアプライアンス
- **Transit Gateway ルートテーブル**
- **関連付け**
  - 各アタッチメントは、正確に 1 つのルートテーブルに関連付けられる
- ルート伝達
  - VPC、VPN 接続、または Direct Connect ゲートウェイは、Transit Gateway ルートテーブルに動的にルートを伝達できる
  - VPC では、Transit Gateway にトラフィックを送信するための静的ルートを作成する必要がある
  - VPN 接続または Direct Connect ゲートウェイでは、ボーダーゲートウェイプロトコル (BGP) を使用してTransit Gateway からオンプレミスのルーターにルートが伝達される
  - ピアリングアタッチメントでは、ピアリングアタッチメントをポイントする静的ルートをTransit Gateway のルートテーブルに作成する必要がある