---
title: "Cloud Interconnect"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

<!--more-->

[Cloud Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/overview?hl=ja)

1. [コンセプト](#1-コンセプト)

## 1. コンセプト

- [Dedicated Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview?hl=ja)
    - オンプレミス ネットワークと Google のネットワークを物理的に直接接続
- [Partner Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/partner-overview?hl=ja)
    - サポート対象のサービスプロバイダを介して、オンプレミス ネットワークと VPC ネットワークを接続
- [ダイレクトピアリング](https://cloud.google.com/network-connectivity/docs/direct-peering?hl=ja)
    - Google Cloud の外部に存在し、 Google Workspace アプリケーションにアクセスする必要がない限り、Google Cloud への推奨のアクセス方法は、Dedicated Interconnect または Partner Interconnect
- [キャリアピアリング](https://cloud.google.com/network-connectivity/docs/carrier-peering?hl=ja)
    - Dedicated Interconnect に対する Partner Interconnect 、ダイレクトピアリングに対するキャリアピアリング
- [CDN Interconnect](https://cloud.google.com/network-connectivity/docs/cdn-interconnect?hl=ja)
    - GCP から Akamai などの CDN プロバイダに対してダイレクトコネクションできる