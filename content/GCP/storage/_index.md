---
title: "ストレージ"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 5
---

<!--more-->

- [Google Cloud オンライン ストレージ プロダクト](https://cloud.google.com/products/storage)
    - [永続ディスク](https://cloud.google.com/compute/docs/disks/add-persistent-disk)

## 1. Google Cloud Storage

AWS でいう S3 。

### 1.1. ロケーションタイプ

- マルチリージョン
    - 選択した地域（ 3 箇所以上のリージョンを含む）に分散してデータを保存
    - `ASIA` 、 `EU` 、 `US` の 3 つに別れているので、 **全世界を対象とする場合、各にバケットを作成する必要がある**
- デュアルリージョン
    - 2 箇所のリージョンに分散してデータを保存
- リージョン
    - 単一リージョンにデータを保存

### 1.2. ストレージクラス

|ストレージクラス|最小保存期間|ユースケース|
|:---|:---|:---|
|Standard|なし|頻繁にアクセスされ短時間だけ保存されるデータ|
|Nearline|30日|月に 1 回程度したアクセスされないデータ|
|Coldline|90日|四半期に 1 回程度したアクセスされないデータ|
|Archive|365日|年に 1 回程度したアクセスされないデータ|

### 1.3. ライフサイクル

古いデータはアーカイブしたり削除したいといったニーズに対応する、オブジェクト管理に関する運用を効率化するための機構。  
複数の条件と、その条件にマッチした際の操作（削除 or ストレージクラスの変更）を設定できる。

### 1.4. バージョニング

説明省略。

### 1.5. `gsutil`

`gsutil` コマンドを利用して管理できる。  
バケットの作成。（ Make Bucket で `mb` ）（ `gs` は google strage ？）

```bash
$ gsutil mb gs://YOUR-BUCKET-NAME/
```

- バケットの名前空間はグローバルなのでグローバルでユニークであり、一般公開されるため、バケット名に機密情報を含めない
- バケット名に使用できる文字は、小文字、数字、ダッシュ（-）、アンダースコア（_）、ドット（.）のみ
    - ドットを使用している名前には [ドメイン名を持つバケットの検証](https://cloud.google.com/storage/docs/domain-name-verification) が必要
    - バケット名の先頭と末尾は、数字または文字にする必要あり
- バケット名の長さは 3～63 文字
    - ドットを使用している名前には最大 222 文字を使用できますが、ドットで区切られている各要素は 63 文字以下
- バケット名はドット区切りの十進表記（たとえば、192.168.5.4）の IP アドレス形式で表すことはできない
- バケット名の先頭に接頭辞「goog」は使用できない
    - バケット名に「google」または「google」と類似する表記を含めることはできない
- DNS の基準に準拠し、将来的な互換性を確保するため、アンダースコア（_）を使用したり、ドットを連続して使用したり、ドットとダッシュを隣同士で使用したりしない
    - たとえば、「..」、「-.」、「.-」などの表記は DNS 名として使用することはできない

ローカルから Cloud Storage バケットへのコピーは以下。

```bash
$ gsutil cp FILE_NAME gs://YOUR-BUCKET-NAME
```

Cloud Storage バケットからローカルへのコピーは以下。（オプション `-r` ）

```bash
$ gsutil cp -r gs://YOUR-BUCKET-NAME/FILE_NAME .
```

image-folder という名前のフォルダを作成し、そこに画像（ada.jpg）をコピーします。

```bash
$ gsutil cp gs://YOUR-BUCKET-NAME/ada.jpg gs://YOUR-BUCKET-NAME/image-folder/
```

一覧表示。（ただし、下の階層までは表示されない）（ `-l` オプションをつけると詳細表示される）

```bash
$ gsutil ls gs://YOUR-BUCKET-NAME
```

オブジェクトを一般公開する。

```bash
$ gsutil acl ch -u AllUsers:R gs://YOUR-BUCKET-NAME/ada.jpg
```

公開を削除する。

```bash
$ gsutil acl ch -d AllUsers gs://YOUR-BUCKET-NAME/ada.jpg
```

オブジェクトの削除。

```bash
$ gsutil rm gs://YOUR-BUCKET-NAME/ada.jpg
```

### 1.6. [再試行方法](https://cloud.google.com/storage/docs/retry-strategy)

Cloud Storage へのリクエストに失敗した場合に再試行するかどうかを決定するには、リクエストのタイプとべき等性を検討する必要がある。

- リトライ
    - 指数バックオフアルゴリズム： `2^n + randam` ： `n` は試行回数
- べき等性
    - 以下のような条件を満たす必要がある
        - オペレーションを連続してリクエストしても、対象リソースに対して結果が生成される。
        - オペレーションが 1 回だけ成功する。
        - 対象リソースの状態に対して観察可能な結果がない。