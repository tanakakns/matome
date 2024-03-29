---
title: "データベース"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 6
---

Cloud SQL、Cloud Spanner、Bare Metal Solution、Cloud Bigtable、Firestore、Memorystore、BigQuery

<!--more-->

[Google Cloud データベース](https://cloud.google.com/products/databases)

|サービス|モデル|提供|ユースケース|備考|
|:---|:---|:---|:---|:---|
|[Cloud SQL](./sql)|リレーショナル（ SQL ）|インスタンス|小規模（TB級）、水平スケーラビリティ不可|[doc](https://cloud.google.com/sql/docs)|
|[Cloud Spanner](./spanner)|リレーショナル（ SQL ）|インスタンス？|大規模（PB級）、水平スケーラビリティ、グローバル分散|[doc](https://cloud.google.com/spanner/docs)|
|[Cloud Bigtable](./bigtable)|Key-Value（ HBase 互換）||大容量（PB級）、超低レイテンシ（レイテンシが許容されるなら BQ ）|[doc](https://cloud.google.com/bigtable/docs)|
|[Firestore ネイティブモード](./firestore-native)|ドキュメント（ドキュメントとコレクション）|サーバレス|TB級、トランザクション可能、サーバレス、水平スケーラビリティ|[doc](https://cloud.google.com/firestore/docs)、[Firebase Realtime Database](https://firebase.google.com/products/realtime-database/)の後継|
|[Firestore Datastore モード](./firestore-datastore)|ドキュメント（エンティティと種類）|サーバレス|TB級、トランザクション可能、サーバレス、水平スケーラビリティ|[doc](https://cloud.google.com/firestore/docs/concepts)、[Datastore](https://cloud.google.com/datastore/docs?hl=ja)の後継|
|[Memorystore](./memorystore)|インメモリ||キャッシュ、超低レイテンシ、非永続化|[doc](https://cloud.google.com/memorystore/docs)|
|[BigQuery](./bigquery)|DWH（ SQL ）||大規模（PB級）なデータアナリティクス|[doc](https://cloud.google.com/bigquery/docs?hl=ja)|

- [ネイティブ モードと Datastore モードからの選択](https://cloud.google.com/firestore/docs/firestore-or-datastore?hl=ja#feature_comparison)
    - モバイルと直接通信の場合はネイティブモード、サーバとの直接通信の場合は Datastore モードが推奨

## データ移行

- gsutil ：オンプレミスデータの転送量が少ない場合
    - Cloud Storage バケットを作成し、データコピー
    - オンプレミスもしくは別の Cloud Storage リージョンからの転送 **1 TB 未満** の場合
- [Storage Transfer Service](https://cloud.google.com/storage-transfer/docs/overview?hl=ja) ：オンプレミスデータの転送量が多い場合
    - 他のクラウドまたはオンプレミスストレージから Cloud Storage バケットにデータを転送またはバックアップするサービス
    - 別の Cloud Storage リージョンからの転送 **1 TB 超** の場合
    - 別のクラウドストレージからの転送の場合
    - オンプレミスから 1 TB 超のデータを転送する場合は [**Transfer service for on-premises data**](https://cloud.google.com/storage-transfer/docs/on-prem-overview?hl=ja)
- Transfer Appliance ：大規模な転送の場合
    - 大量のデータ（数百テラバイトから 1 ペタバイトまで）を Google Cloud Platform に安全に移行できるハードウェア アプライアンス
    - トータルの所用期間は 50 日
- BigQuery Data Transfer Service ：あらかじめ設定されたスケジュールに基づき、BigQuery へのデータの移動を自動化するマネージド サービス

## Firestore Datastore モード

### コンセプト

- エンティティ
    - エンティティは 1 つ以上の名前付きプロパティを持ち、各プロパティが 1 つ以上の値を持つ
    - 同じ種類のエンティティが同じプロパティを持つとは限らない
    - エンティティの特定のプロパティの値がすべて同じデータ型である必要はない
- プロパティ
    - 整数、浮動小数点数、文字列、日付、バイナリデータ のデータ型をサポート
- キー
- 祖先キー
    - そのエンティティの親となるエンティティの主キー
    - こうすることで、エンティティがツリー構造になる
    - このツリーを **エンティティグループ** という
- エンティティグループ
    - 祖先キーとキーから構成されるツリー構造となったエンティティ群
    - エンティティグループに対する書き込み処理は1秒間1回しか実行できないが、一貫性のあるクエリを実現できる
        - Datastore の提供するクエリは基本的に 結果整合のクエリ だが、エンティティグループにおいては例外で、これがエンティティグループを作る理由

RDBMS と比較した場合は以下の対応になる。

|RDBMS|Datastore|
|:---:|:---:|
|スキーマ（データベース）|ネームスペース|
|テーブル|カインド（種類）|
|レコード|エンティティ|
|プライマリキー|キー + 祖先キー|

### 操作

Go 言語の場合で記載する。

```go
# エンティティの作成
type Task struct {
        Category        string
        Done            bool
        Priority        float64
        Description     string `datastore:",noindex"`
        PercentComplete float64
        Created         time.Time
}
task := &Task{
        Category:        "Personal",
        Done:            false,
        Priority:        4,
        Description:     "Learn Cloud Datastore",
        PercentComplete: 10.0,
        Created:         time.Now(),
}

# エンティティの登録・更新
key := datastore.IncompleteKey("Task", nil)
key, err := client.Put(ctx, key, task)

# エンティティの取得
var task Task
taskKey := datastore.NameKey("Task", "sampleTask", nil)
err := client.Get(ctx, taskKey, &task)

# エンティティの削除
key := datastore.NameKey("Task", "sampletask", nil)
err := client.Delete(ctx, key)

# 一括オペレーション（バッチ）
tasks := []*Task{
        {
                Category:    "Personal",
                Done:        false,
                Priority:    4,
                Description: "Learn Cloud Datastore",
        },
        {
                Category:    "Personal",
                Done:        false,
                Priority:    5,
                Description: "Integrate Cloud Datastore",
        },
}
keys := []*datastore.Key{
        datastore.IncompleteKey("Task", nil),
        datastore.IncompleteKey("Task", nil),
}
keys, err := client.PutMulti(ctx, keys, tasks)
# その他に GetMulti 、 DeleteMulti もある
```

### エンティティのエクスポートとインポート

バックアップなどのために [エンティティのエクスポートとインポート](https://cloud.google.com/datastore/docs/export-import-entities) を実行する場合、以下の方法で **マネージドエクスポート・インポートサービス** を利用できる。

- Cloud Console
- gcloud コマンドライン ツール
- Datastore Admin API（REST、RPC）

ただ、スケジューラ機能は無いので、 cron ジョブで実行するなどが必要になる。

## Bigtable

### Bigtable インスタンスの変更

以下の場合、ダウンタイム無しで設定を変更できる。

- 各クラスタのノードの数
- インスタンス内のクラスタの数
- インスタンスのアプリ プロファイル（レプリケーション設定が含まれている）
- インスタンスのラベル（インスタンスに関するメタデータを提供する）
- インスタンスの表示名
- 開発インスタンスから本番インスタンスへのアップグレード

ただし、以下を変更する場合は、Bigtable インスタンスを新たに作成し、データを移行する必要がある。

- インスタンス ID
- ストレージ タイプ（SSD または HDD）
- 顧客管理の暗号鍵（CMEK）の構成

移行の方法としては、Dataflowを使用して古いBigTableインスタンスから新しいインスタンスにデータを移行するなどがあげられる。

### ストレージの選択

Bigtable では HDD または SSD のストレージを選択することができる。  
選択基準は以下の通り。

- HDD
    - 非常に大きいデータセット（10 TB 超）で、レイテンシがあまり重要でない場合やアクセス頻度が低い場合に適切
    - バッチやデータアーカイブなど
- SSD
    - ほとんどのユースケースで最も効率的でコスト効果の高い選択肢

### レプリケーション

クラスタ単位のレプリケーションをするみたい。驚き。（ [参考](https://cloud.google.com/bigtable/docs/replication-overview) ）

### パフォーマンステスト

Bigtable でパフォーマンステストを実施する場合は以下の点に留意する。

- 十分なデータでテストする
    - ノードあたり 100 GB 以上
        - ただし、 3 ノード以上でクラスタ化されているので 300 GB 以上
- 単一のテーブルでテストする
- ノードあたりの推奨ストレージ使用率を超えないように
- テスト前に、負荷の大きな事前テストを数分間実行
    - データ配置が最適化される
- 最低 10 分間テストを実行
    - データをさらに最適化するので、ディスクからの読み取りだけでなく、メモリからのキャッシュ経由の読み取りも確実にテストできる