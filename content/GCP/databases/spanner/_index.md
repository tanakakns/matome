---
title: "Cloud Spanner"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

<!--more-->


## トランザクション

大きく以下の 3 タイプの [トランザクション](https://cloud.google.com/spanner/docs/transactions) をサポートする。

- ロック型読み取り / 書き込み
    - 書き込みトランザクション
    - 悲観ロックに依存し、必要に応じて 2 フェーズ commit される
- 読み取り専用
    - 読み取りトランザクション
    - commit する必要がなく、ロックされることもない
- パーティション化 DML
    - 一括更新・削除トランザクション
    - 定期的なクリーンアップやバックフィルに適する

### 読み取り

[読み取り](https://cloud.google.com/spanner/docs/reads) には以下のタイプがある。

- 強力な読み取り（デフォルト）
    - 現在のタイムスタンプでの読み取り
    - 読み取りの開始時までに commit されたすべてのデータを確実に取得
- ステイル読み取り
    - 過去のタイムスタンプによる読み取り
    - レイテンシの影響を受けやすいものの古いデータは許容できる場合、パフォーマンスが向上することがある

## 参考

- [ゲーム データベースとして Cloud Spanner を使用する場合のベスト プラクティス](https://cloud.google.com/architecture/best-practices-cloud-spanner-gaming-database)