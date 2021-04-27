---
title: "監視"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 8
---

<!--more-->


## Stackdriverの種類

- Monitoring
    - アラート定義
    - アップタイムチェック、稼働時間モニタリング
    - AWSもまとめてみれる
- Logging
    - ログに基づいた指標定義
    - フィルターや検索でアプリケーションログを見る
    - BigQuery、Google Cloud Storage、Pub/Subへのエクスポート
    - エージェント導入できる
- Trace
    - Google App Engine 用のレイテンシのサンプリング
    - URL ごとの統計
    - レイテンシの分布
- Debugger
    - 本番環境でのコードのデバッグ（途中で止める）
- Error Reporting
    - エラーをレポート。通知