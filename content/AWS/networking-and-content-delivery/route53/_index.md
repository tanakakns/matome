---
title: "Amazon Route 53"
date: 2023-06-27T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 5
---

Amazon Route 53 は、可用性と拡張性に優れたドメインネームシステム (DNS) ウェブサービス。

<!--more-->

# 概要

Amazon Route 53 は以下の 3 つの機能を持つ。

- ドメイン名の登録
- ドメインのリソースへのインターネットトラフィックのルーティング
- リソースの正常性のチェック

# ルーティング

## ルーティングポリシー

- シンプルルーティングポリシー
  - 標準のDNSレコードの設定によるルーティング
- フェイルオーバールーティングポリシー
  - リソースが正常な場合リソースにトラフィックをルーティング、異常な場合は別のリソースにトラフィックをルーティング
- 位置情報ルーティングポリシー
  - ユーザーの地理的場所、つまり DNS クエリの送信元の場所に基づいて、トラフィックを処理するリソースを選択できる
- 地理的近接性ルーティングポリシー
  - ユーザーとリソースの地理的場所に基づいてリソースのトラフィックをルーティング
- レイテンシールーティングポリシー
  - 複数の AWS リージョンでアプリケーションがホストされている場合、ネットワークレイテンシーが最も低い AWS リージョンに基づいてリクエストを処理することで、ユーザーのパフォーマンスを向上させることができる
- IP ベースのルーティングポリシー
  - IP ベースのルーティングでは、ユーザー IP からエンドポイントにマッピングする形で Route 53 にデータをアップロードするので、パフォーマンスの最適化やネットワークコスト削減のための、きめ細かな制御が可能になる
- 複数値回答ルーティングポリシー
  - DNS クエリに対する応答として複数の値 (ウェブサーバーの IP アドレスなど) を返すように設定できる
- 加重ルーティングポリシー
  - 単一のドメイン名 (example.com) またはサブドメイン名 (acme.example.com) に複数のリソースを関連付け、各リソースにルーティングされるトラフィックの数を選択できる
  - 負荷分散や新しいバージョンのソフトウェアのテストなど、さまざまな目的に有用

# Route 53 Resolver

Amazon VPC 固有の DNS リゾルバで IP アドレス「x.x.x.2」として存在する。  
※ DNS リゾルバ・・・ユーザのリクエストと DNS サーバーを仲介するサーバ。

Route 53 Resolver の **インバウンドエンドポイント** は、VPC **外部** からVPC内の名前解決を可能にするためのエンドポイント。  
Route 53 Resolver の **アウトバウンドエンドポイント** は、VPC **内部** からVPC内の名前解決を可能にするためのエンドポイント。