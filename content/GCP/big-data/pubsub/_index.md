---
title: "Cloud Pub/Sub"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

<!--more-->

[Cloud Pub/Sub](https://cloud.google.com/pubsub/docs/concepts)


## 基本情報

Google Cloud Pub/Sub は非同期グローバル メッセージング サービス。  
Pub/Sub では、 **topics** 、 **publishing** 、 **subscribing** という 3 つのキーワードが頻出する。

- topic
    - 複数のアプリケーションが共通のスレッドを通じて相互に接続できる共有文字列
- パブリッシャー
    - メッセージを Cloud Pub/Sub トピックに push（publish）する
- サブスクライバー
    - そのスレッドへの「subscription」を作成し、トピックからメッセージを pull するか、push サブスクリプション用の webhook を構成する
    - すべてのサブスクライバーは、構成可能な時間の範囲内で各メッセージに確認応答を返信する必要がある

まとめると、パブリッシャーはメッセージを作成してトピックに送信し、サブスクライバーはトピックへのサブスクリプションを作成してトピックからメッセージを受け取る。

以下のコマンドで `myTopic` というトピックを管理できる。

```bash
# トピックの作成
$ gcloud pubsub topics create myTopic

# 確認
$ gcloud pubsub topics list

# 削除
$ gcloud pubsub topics delete myTopic
```

次のコマンドを実行して、トピック `myTopic` へのサブスクリプション `mySubscription` を作成する。

```bash
# サブスクリプションの作成
$ gcloud pubsub subscriptions create --topic myTopic mySubscription

# 確認
$ gcloud pubsub topics list-subscriptions myTopic

# 削除
$ gcloud pubsub subscriptions delete mySubscription
```

作成したトピック `myTopic` にメッセージ `hello` をパブリッシュする。

```bash
# メッセージのパブリッシュ
$ gcloud pubsub topics publish myTopic --message "Hello"

# サブスクリプション経由でメッセージを pull
$ gcloud pubsub subscriptions pull mySubscription --auto-ack
┌───────┬──────────────────┬──────────────┬────────────┬──────────────────┐
│  DATA │    MESSAGE_ID    │ ORDERING_KEY │ ATTRIBUTES │ DELIVERY_ATTEMPT │
├───────┼──────────────────┼──────────────┼────────────┼──────────────────┤
│ Hello │ 2373143694323857 │              │            │                  │
└───────┴──────────────────┴──────────────┴────────────┴──────────────────┘
```

- `--auto-ack` フラグを指定せずに pull コマンドを使用すると、メッセージが 1 つだけ出力される。これは、サブスクライブしているトピックに複数のメッセージがある場合も同じ。
- 特定のサブスクリプション ベースの pull コマンドから 1 つのメッセージが出力された後は、pull コマンドを使って同じメッセージに再びアクセスすることはできない。

数量を指定して pull する場合は以下の通り。

```bash
$ gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
```