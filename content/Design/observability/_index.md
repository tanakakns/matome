---
title: "Observability（O11y：可観測性）"
date: 2023-06-20T12:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 5
---

Observability（O11y：可観測性）についてまとめる。

<!--more-->

## Observabilityとは？

**Observability** （ **O11y** : **可観測性** ）とは「Observe（観察する）」と「ability（能力）」を組み合わせて、観察する能力を意味する造語。  
つまり『 __いつ、どこで、何が、なぜ起こっているのかを観測可能に保つ__ 』考え方。

## Observability（可観測性）とMonitoring（監視）の違い

一言でいうと、監視はシステムのオブザーバビリティを向上させるために行う活動のこと。  

これまでのMonitoring（監視）は主に基盤的な観点が中心で、プロセス監視やメトリック監視（CPUやメモリ）、閾値で監視を行い、アラートがあがり、問題があったあとに対応を行う。  
つまり、何らかの問題が発生してから対応することになります。

基盤以外のアプリケーションレイヤにまで踏み込んだものが **APM** （ **Application Performance Management** ）。  
アプリケーションのゴールデンシグナル（スループット、エラー発生率、応答性能など）を中心に、サービスの提供状況を理解するアプローチ。

Observabilityは、これらMonitoringやAPMのアプローチをベースとして、システムの複数層のデータをリアルタイムで解析し、複雑化したシステムにおいて 問題点を「積極的に」理解し、問題状況の把握をおこなうアプローチだと言える。  
Monitoringの「問題が起きた後に調べれば分かる」に対して、『 __いま起きていることをデータに基づいて説明できる、改善できる__ 』状況に持っていくものが Observability だと言える。

## Observabilityの構成要素

- **メトリクス** : 何が起きているのか
  - 特定の時間間隔で測定して統計したもの
  - CPU使用率などのリソース状況や、レイテンシなどのサービス状況といった、定量的な指標となる数値データ
- **ログ** : 何が起きたのか
  - 発生したイベントをタイムスタンプで記録したもの
  - OSやミドルウェア、アプリケーションなどが出力する、個別のイベント情報
- **トレース** : どこで起きたのか
  - 依存関係のある一連のリクエストフローを始まりから終わりまで変換したもの
  - アプリケーション処理で発生するリクエスト構造や、呼び出される各コンポーネントでの処理時間などを可視化したもの

上記は Observability で収集する **テレメトリーデータの「3本柱」** と言われている。  
他には以下がある。

- **プロファイラ**
  - アプリケーション稼働時に、アプリケーションがどのように実行され、リソースがどこでどの程度割り当てられていたのかを示す情報
  - CPUプロファイラ、メモリ割当プロファイラ、ヒーププロファイラなどがある
- **ダンプファイル（コアダンプ）**
  - システムがクラッシュした際に出力されるメモリイメージ

### メトリクス

収集メトリクスには以下のようなものがある。

1. **USE** ：リソースをベースとしたメトリクス収集
  - **U** tilization：リソースの単位時間あたりの使用率（CPU、メモリ使用率、など）
  - **S** aturation：リソースの飽和状態（キューの長さ、など）
  - **E** rror：エラーイベントのカウント（Network,Disk I/O エラー、など）
2. RED：サービスをベースとしたメトリクス収集
  - **R** ate（Request）：秒間のリクエスト数
  - **E** rror：失敗しているリクエスト数
  - **D** uration：リクエスト処理に要した時間
3. **The Four Golden Signals**
  - **Latency** ：リクエストを処理するのに要した時間
  - **Traffic** ：システムに対するリクエスト量（リクエスト数、Network I/O数、セッション数、など）
  - **Error** ：処理の失敗
  - **Saturation** ：サービスが手一杯になった状態（CPU、メモリ、ディスク、Network I/O、など）

### ログ

ログには以下のようなものがある。

- **アプリケーションログ**
  - アプリケーションが出力するログ
- **システムログ**
  - OS、ミドルウェア、システムコンポーネントが出力するログ
- **監査ログ**
  - AWS APIなどのシステム変更管理用のAPI、RDBへのアクセスなど、何らかの権限を有した操作のログ

### トレース

トレース対象としては以下のようなものがある。

- 分散トレーシング
  - マイクロサービスやコンポーネントをまたがるリクエストの追跡
- スパン
  - リクエストの開始〜終了までに１サービス、あるいは複数サービスで実行された処理の単位（スパン）で監視する
- エンドユーザ監視
  - エンドユーザのパフォーマンスを監視する
- 合成モニタリング
  - サービスの死活監視をする、外形監視という場合もある

## Observabilityを実現するテクノロジ

OSS、AWSでは以下のような感じ。

|＼      |OSS                               |AWS                                                                               |
|:-------|:--------------------------------|:---------------------------------------------------------------------------------|
|メトリクス|Prometheus & Grafana             |CloudWatch Metrics、Amazon Managed Grafana & Amazon Managed Service for Prometheus|
|ログ     |EFK(ElasticSearch,Fluentd,Kibana)|CloudWatch Logs                                                                   |
|トレース  |Jaeger、Kiali、OpenTelemetry     |X-Ray                                                                             |

OpenTelemetryは現状はアプリケーションのオブザーバビリティのためのライブラリ群で、現状はトレースとメトリクスに関するライブラリを提供している。（[参考](https://ymotongpoo.hatenablog.com/entry/2020/06/01/164221)、[参考2](https://zenn.dev/yuta28/articles/what-is-opentelemetry)）  
OpenMetrics は Prometheus が利用しているフォーマットを仕様として抽出したもの。

上記のOSSやパブリッククラウドの機能だけでは APM レイヤがカバーしきれていない。（ Elastic APM というのもあるにはあるが、、、）  
そのため、 APM/Observability の製品を検討する必要がある。

- Datadog
  - Gartner Magic Quadrant for APM and Observability で 1 or 2番手のリーダー（ [参考](https://www.dynatrace.com/ja/gartner-magic-quadrant-for-application-performance-monitoring-observability/) ）
  - CNCF End User Technology Radar の Observability, September 2020 で ADOPT （ [参考](https://radar.cncf.io/2020-09-observability) ）
- Dynatrace
  - Gartner Magic Quadrant for APM and Observability で 1 or 2番手のリーダー（ [参考](https://www.dynatrace.com/ja/gartner-magic-quadrant-for-application-performance-monitoring-observability/) ）
- NewRelic
  - Gartner Magic Quadrant for APM and Observability で 3番手のリーダー（ [参考](https://www.dynatrace.com/ja/gartner-magic-quadrant-for-application-performance-monitoring-observability/) ）
- Elastic
  - Gartner Magic Quadrant for APM and Observability で 7番手くらいのビジョナリー（ [参考](https://www.dynatrace.com/ja/gartner-magic-quadrant-for-application-performance-monitoring-observability/) ）
  - CNCF End User Technology Radar の Observability, September 2020 で ADOPT （ [参考](https://radar.cncf.io/2020-09-observability) ）



- [Datadog vs Dynatrace vs New Relic comparison](https://www.peerspot.com/products/comparisons/datadog_vs_dynatrace_vs_new-relic)
- [Compare New Relic vs Dynatrace vs Datadog](https://crozdesk.com/compare/new-relic-vs-dynatrace-vs-datadog)