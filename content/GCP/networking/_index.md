---
title: "ネットワーキング"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 7
---

1. VPC

<!--more-->

- [Google Cloud ネットワーキング プロダクト](https://cloud.google.com/products/networking)
- 接続
    - [Virtual Private Cloud（VPC）](https://cloud.google.com/vpc/docs)

## 1. VPC

Virtual Private Cloud（VPC）は、Compute Engine 仮想マシン（VM）インスタンス、Google Kubernetes Engine（GKE）クラスタ、[App Engine フレキシブル環境](https://cloud.google.com/appengine/docs/flexible) にネットワーキング機能を提供する。

### 1.1. VPC ネットワーク

- VPC ネットワークは、データセンター内のリージョン仮想サブネットワーク（ **サブネット** ）のリストで構成されるグローバルリソースであり、すべてグローバルな広域ネットワークで接続される。
- **ルート**、**ファイアウォール** ルールを含む VPC ネットワークは **グローバルリソース** 。特定のリージョンやゾーンには関連付けられていない。
- サブネットは [リージョンリソース](https://cloud.google.com/compute/docs/regions-zones/global-regional-zonal-resources#regionalresources) 。サブネットごとに IP アドレスの範囲が定義される。（ VPC には範囲が定義されない）