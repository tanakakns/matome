---
title: "機械学習"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 10
---

Cloud Machine Learning Engine

<!--more-->

1. Cloud Machine Learning Engine

## 1. Cloud Machine Learning Engine

TensorFlow の実行環境を提供するサービスであり、「トレーニング」と「予測（「オンライン予測」と「バッチ予測」）」の両方をサポートする。  
マシンタイプ以外には以下が課金対象となる。

- ノードによるバッチ予測ジョブの処理時間
- ノードによるオンライン予測リクエストの処理時間
- ノードがオンライン予測の準備状態になっている時間

### 1.1. オンライン予測 と バッチ予測

[参考](https://cloud.google.com/ai-platform/prediction/docs/online-vs-batch-prediction)

### 1.2. カスタムクラスタ

[カスタムクラスタ](https://cloud.google.com/ai-platform/training/docs/machine-types) に設定できるパラメータは以下。

- workerType、parameterServerType、evaluatorType、workerCount、parameterServerCount、evaluatorCount

## 2. AutoML

以下の総称？

- AutoML Vision
- AutoML Video Intelligence
- AutoML Natural Language
- AutoML Translation
- AutoML Tables

### AutoML Visionのモデルの反復

以下の手順にある [モデルの反復](https://cloud.google.com/vision/automl/docs/evaluate#iterate_on_your_model) により精度を向上させる。

- AutoML Vision では、モデルの「混同」の度合い、真のラベル、予測されたラベルによって、画像を並べ替えることができます。これらの画像に目を通し、適切にラベルが付けられていることを確認してください。
- 低品質のラベルにさらに画像を追加することを検討してください。
- さまざまなタイプの画像を追加しなければならない場合があります（たとえば、構図の拡大、解像度の増減、異なる視点など）。
- 十分なトレーニング画像が存在しない場合は、 **ラベルを完全に削除** することを検討してください。
- マシンにはラベル名の意味は理解できません。つまり、ラベル名はマシンにとって単なるランダムな文字列にすぎません。「door」というラベルと「door_with_knob」という別のラベルがある場合、マシンにはそのラベルが付いた動画以外にニュアンスを把握する方法はありません。
- 真陽性と真陰性のサンプルを追加してデータを強化してください。特に重要なサンプルは、意思決定の境界に近いものです（混同を招く可能性があるが、正しくラベル付けされているもの）。
- 独自の TRAIN、TEST、VALIDATION の分割を指定してください。ツールは画像をランダムに割り当てますが、割り当てが重複に近い場合は TRAIN と VALIDATION の過学習につながり、TEST セットのパフォーマンスが低下する可能性があります。