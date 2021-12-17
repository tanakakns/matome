---
title: "API設計"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 2
---

1. REST API

<!--more-->

## 1. REST API

個人的に REST API の設計をまとめる。  

### 基本方針

|処理の内容|HTTPメソッド|URL|プログラムメソッド|
|:---|:---|:---|:---|
|リソースデータを全て取得する|GET| `~/resources` | `ListResources` |
|idと一致するリソースを一件取得する            |GET          | `~/resources/{id}` | `GetResource` |
|リソースを一件登録する                      |POST         | `~/resources` | `CreateResource` |
|idと一致するリソースを一件更新する            |PUT          | `~/resources/{id}` | `UpdateResource` |
|idと一致するリソースを一件更新する(ただし提供されたフィールドのみ)|PATCH        | `~/resources/{id}` | `UpdateResource` |
|idと一致するリソースを一件削除する            |DELETE       | `~/resources/{id}` | `DeleteResource` |
|リソースの利用可能なメソッドとパラメータを返す|OPTIONS      | `~/resources` | - |

- URL は **ケバブケース**
  - NG : `/systemOrders` 、 `/system_orders`
  - OK : `/system-orders`
- クエリパラメータは **キャメルケース** 、JSON のプロパティにも **キャメルケース**
  - NG : `/system-orders/{order_id}` 、 `/system-orders/{OrderId}`
  - OK : `/system-orders/{orderId}`
- URL中の `resources` はリソースを表し、コレクションつまり **複数形の名詞** で表現
  - NG : `GET /user` 、 `GET /User`
  - OK : `GET /users`
- URL はコレクションで始まり、識別子で終わる
  - NG : `GET /shops/:shopId/category/:categoryId/price`
  - OK : `GET /shops/:shopId/` 、 `GET /category/:categoryId`
- リソース URL に動詞を使わない
  - NG : `POST /updateuser/{userId}` 、 `GET /getusers`
  - OK : `PUT /user/{userId}`
- 非リソース URL は動詞を使う
  - OK : `POST /alerts/245743/resend`
  - リソースへの CRUD ではなく、特定のビジネスロジックを実行する場合
- `/health` 、 `/version` 、 `/metrics` の各 API エンドポイントを実装する
  - `/health` : ヘルスチェック
  - `/version` : バージョン番号
  - `/metrics` : 平均応答時間などのさまざまなmetricsを提供
  - `/debug` と `/status` もおすすめ
- リソース名にテーブル名を使わない
  - NG : `product_order`
  - OK : `product-orders`
- レスポンスにリソースの総数を含める

```javascript
{
    users: [
        ...
    ],
    total: 34 # これ
}
```

- ゴールデンルール
  - フラットはネストより良い
  - 単純は複雑より良い
  - 文字列は数字より良い
  - 一貫性はカスタマイズより良い

### クエリパラメータの使用

- 検索条件を指定したい場合にしようする
  - SELECT文のWHERE句のように使用する
  - 例）`~/dogs?age=3`
    - 3歳の犬だけを取得する
- ある1つのリソースの一部の項目だけを取得したい場合（Partial response）
  - 「 **fields** 」にカンマ区切りで指定
  - 例） `~/dogs/1234?field=name,color,location`
- あるリソースから件数を指定して取得したい場合（Pagination）
  - 「 **limit** 」と「 **offset** 」で指定する
  - 「offset」番目から「limit」件取得する
  - 例）`~/dogs?offset=25&limit=50`
  - 全レコード件数を「metadata」としてレスポンスに含める

### リソース操作

計算・翻訳・変換など、リソースへのCRUD以外の操作も存在し得る。  
その場合は名詞ではな **動詞** を使用する。（（例）~/convert?from=EUR&to=CNY&amount=100）  
このAPIはあくまで特殊なケースであることを明記・認識する必要がある。  
また、このAPIで取得できるデータはデータベースの値を直接返すのではなく、ロジックで計算して得た結果であるということもまた明記・認識する必要がある。

### コンテンツネゴシエーション

Restful APIにおいてクライアントからサーバへレスポンスのフォーマットを要求することをコンテンツネゴシエーションという。  
クライアントからサーバへコンテンツのフォーマットを要求する方法として以下の３つがある。

- URLの拡張子
  - 例）HTML形式の要求
    - `https://myserver/myapp/accounts/list.html`
  - 例）spreadsheet形式の要求
    - `https://myserver/myapp/accounts/list.xls`
- リクエスト（クエリ）パラメータ
  - 例）spreadsheet形式の要求
    - `https://myserver/myapp/accounts/list?format=xls`
    - `https://myserver/myapp/accounts/list?mediaType=xls`
- Acceptヘッダ
  - 例）JSON形式の要求
    - `Accept: application/json`

個人的には **Accept ヘッダ** 推奨。  
「Accept-Language」「Accept-Encoding」も併せて利用する。

### 設計ステップ

1. 対象となるデータを洗い出す
  - ER図とか書くんじゃね
2. 洗い出したデータをリソースに分割する
  - ER図の1エンティティを1リソースにしたり、結合したテーブルを1リソースにしたり
3. リソースにURLを割りあてる
  - リソースは **名詞の複数形** にすること
    - 例外的に動詞を使用する場合がある（後述）
  - URLが右に行くほど自然な階層構造・サブセットになっているか
  - 例）オーナー5678が飼っている犬を表す場合
    - `~/owners/5678/dogs`
    - `~/resources/{id}/resources/{id}` 以上に階層を深くしない
    - これ以上複雑にしたい場合はクエリパラメータを使用する
4. リソースに対してインターフェースを割り当てる
  - HTTPメソッド（GET、POST、PUT、DELETE等）の選択
5. クライアントから受信するメディアタイプを1つ以上決める
  - `application/json` 、 `application/xml` 、等
6. クライアントへ返信するメディアタイプを1つ以上決める
  - `application/json` 、 `application/xml` 、等
7. 接続性（Connectedness）を高める
  - リソースはハイパーメディアリンクによって他のリソースと接続することができる
  - nクライアントは自らURLを組み立てることは行わず、サーバから送られてくるハイパーメディアリンクを利用する
8. 正常系を考える（適切なリクエストがあったとき何が起こるか
  - ステータスコードの選択
9. 例外条件を考える（不適切なリクエストがあったとき何が起こるか
  - ステータスコードの選択
  - ステータスコードと例外をマッピングする

ステータスコードは概ね下記のものを検討すればよい。

|HTTP Status Code        |意味                                                           |発生し得る契機          |
|:---|:---|:---|
|200 OK                   |正常に処理が完了                                                |GET、PUT、DELETEの結果   |
|201 Created              |新規リソースが作成された                                        |POSTの結果               |
|204 No Content           |正常に完了したが、特に返却するBodyが無い                        |POST、DELETEの結果       |
|301 MovedPermanently     |アクセスしたリソースが別のURIに恒久的に移動した                 |リソースのURIが変わった時|
|                         |Locationヘッダに移動先のURIを付与する                           |                         |
|303 See Other            |別のURIにアクセスしてほしい                                     |アクセスURIを変えてほしい|
|304 Not Modified         |リソースの変更をしなかった                                      |POST、PUT、DELETEの結果  |
|307 Temporary Redirect   |一時的なリダイレクト                                            |閉塞時等                 |
|                         |Locationヘッダにリダイレクト先のURIを付与する                   |                         |
|400 Bad Request          |クライアントからのHTTPリクエストに誤りがありサーバで処理できない|                         |
|401 Unauthorized         |クライアントが認証されていない                                  |                         |
|403 Forbidden            |クライアントの権限不足                                          |                         |
|404 Not Found            |アクセスしたURIが存在しない                                     |                         |
|406 Not Acceptable       |リクエストのAcceptヘッダがサーバで受け入れられない              |コンテンツネゴシエーション失敗|
|415 Unsupported MediaTYpe|リクエストのContent-Typeヘッダがサーバで受け入れられない        |サーバ側でBodyを解釈できない|
|409 Conflict             |リソースの現状の状態の矛盾している                              |更新順序の前後等|
|500 Internal Server Error|サーバ内でエラーが発生した                                      |上記以外のサーバエラー|

### レスポンスの設計

|API処理の結果            |レスポンスStatus Code   |レスポンスBody                                        |
|:---|:---|:---|
|リソースの取得に成功      |200 OK                   |取得対象データ                                         |
|リソースの登録に成功      |201 Created              |追加したデータ（Locationヘッダには追加したデータのURL）|
|リソースの更新・削除に成功|200 OK                   |空                                                     |
|リソースが存在しない      |404 Not Found            |空                                                     |
|リクエストに検証エラー    |400 Bad Request          |エラー情報                                             |
|サーバ側でエラー          |500 Internal Server Error|エラー情報                                             |

### APIの表し方

そのURLがAPIであることを表す方法は大きく2つある。

1. サブドメインを置く
  - 例） `https://api.example.com/`
2. パスに置く
  - 例） `https://example.com/api/`

サブドメインが多数派らしい（？）。  
ちなみにAPIをパスで表現していて、且つ、同じドメインでWebページを公開しているサービスの場合、 `https://example.com/web/` のようにするサービスがある模様。

### バージョンの表し方

APIが仕様変更した際に既存クライアントに影響を与えないようにURLにバージョンを指定するサービスが多い。  
その際、以下の観点がある。

- 「v」をつけるorNot
  - 例） `http://example.com/api/v2/`
  - 例） `http://example.com/api/2/`
- 整数or少数
  - 例） `http://example.com/api/2/`
  - 例） `http://example.com/api/2.0/`

「v有」「整数」（ `http://example.com/api/v2/` ）が多いようだ。

### その他、決める際に困りそうなこと

答えは出てないがメモっとく。

- リソースの移動、コピー
- 複数レコードを選択して更新するＩＦ
  - 207 Multi-Statusで返したくなっちゃう

### Web画面の基本方針

|処理の内容     |HTTPメソッド|URL                       |
|:---|:---|:---|
|一覧画面表示    |GET          |~/web/resources            |
|詳細画面表示    |GET          |~/web/resources/{id}       |
|追加Form表示    |GET          |~/web/resources/new        |
|追加アクション  |POST         |~/web/resources/new        |
|編集フォーム表示|GET          |~/web/resources/{id}/edit  |
|編集アクション  |POST         |~/web/resources/{id}/edit  |
|削除画面表示    |GET          |~/web/resources/{id}/delete|
|削除アクション  |POST         |~/web/resources/{id}/delete|

Railsっぽい。

### 参考

- [Cloud APIs API 設計ガイド](https://cloud.google.com/apis/design/)
- [API設計スキルを次のレベルに引き上げるベストプラクティス22選](https://qiita.com/baby-degu/items/6f516189445d98ddbb7d)
- [Zalando RESTful API と イベントスキーマのガイドライン](https://restful-api-guidelines-ja.netlify.app/)
  - Original: [Zalando RESTful API and Event Guidelines](https://opensource.zalando.com/restful-api-guidelines/)