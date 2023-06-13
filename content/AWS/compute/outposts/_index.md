---
title: "AWS Outposts"
date: 2023-06-09T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 5
---

<!--more-->

[AWS Outposts](https://docs.aws.amazon.com/ja_jp/outposts/)についてまとめる。

- 1. [概要](#1-概要)

## 1. 概要

**AWS Outposts** は、自社のデータセンター・拠点にAWSのラック・サーバーを設置して、AWSを自社のオンプレミスサーバーのように拡張して使えるサービス。  
AWSのクラウドと超低レイテンシーで接続できるなどのメリットが生まれる。

- 自社データセンターの中にAWSラックを配置
  - そのサーバはVPC内
  - つまり、AWSとの超低レイテンシー接続が可能
- 所有・管理はAWS側
- AWSのVPCを自社データセンターに拡張する感覚で使える
- ただし、「AWS Direct Connectを用いた専用線接続」か「VPNを用いたインターネット経由接続」が前提