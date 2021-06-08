---
title: "アーキテクチャ"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

[Cloud アーキテクチャセンター](https://cloud.google.com/architecture?hl=ja)

1. [デプロイ戦略](#1-デプロイ戦略)

<!--more-->

## 1. デプロイ戦略

[参考](https://cloud.google.com/architecture/application-deployment-and-testing-strategies?hl=ja)

|パターン|メリット|デメリット|説明|
|:---|:---|:---|:---|
|再作成|シンプル|ダウンタイム発生||
|ローリングアップデート|ダウンタイム無|ロールバック遅、2バージョン両立時の下位互換性、スティッキーセッション||
|Blue/Green|ダウンタイム無、即時ロールバック、環境分離|コスト、下位互換性、カットオーバー|両立せず即時切り替え|
|カナリア|ダウンタイム無、高速ロールバック、本番テスト|リリース遅、堅牢な監視必要、下位互換性、スティッキーセッション||
|A/B テスト||||