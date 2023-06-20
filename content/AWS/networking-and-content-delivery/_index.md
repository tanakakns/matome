---
title: "ネットワーキングとコンテンツ配信"
date: 2023-05-30T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 17
---

ToDo

<!--more-->

{{% children description="true"   %}}

- **Gateway Load Balancer**
  - 受信したトラフィックを複数のアベイラビリティーゾーンの複数のターゲット (Elastic Load Balancing) に自動的に分散することができるサービス
  - 登録されているターゲットの状態をモニタリングし、正常なターゲットにのみトラフィックをルーティングする