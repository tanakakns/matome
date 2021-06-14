---
title: "開発ツール"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 2
---

<!--more-->

1. Cloud Deployment Manager
2. Cloud Build
3. Cloud Tasks

## 1. Cloud Deployment Manager

AWS でいう CloudFormation 。  
単一の API に対応した **リソース** とそのリソースをまとめた **デプロイメント** などで構成される。  
`gcloud deployment-manager deployments` コマンドでデプロイメントを管理し、 `gcloud deployment-manager resouces` コマンドでデプロイメントに含まれるリソースを管理する。

## 2. Cloud Build

[doc](https://cloud.google.com/build/docs/concepts?hl=ja)

**ビルド構成** を作成して、実行するタスクを Cloud Build に指示できる。  
依存関係をフェッチして、単体テスト、静的分析、結合テストを実行し、docker、gradle、maven、bazel、gulp などのビルドツールでアーティファクトを作成するようにビルドを構成できる。  
Cloud Build は、一連の **ビルドステップ** としてビルドを実行し、各ビルドステップは、 *Docker コンテナで実行* される。ビルドステップの実行は、スクリプトでコマンドを実行する場合と似ている。  
Cloud Build や Cloud Build コミュニティで提供されているビルドステップを使用することも、独自のカスタム ビルドステップを作成することもできる。

- Cloud Build が提供するビルドステップ
    - 一般的な言語とタスク用にサポート対象のオープンソース ビルドステップが用意されている
- コミュニティ提供のビルドステップ
    - Cloud Build のユーザー コミュニティで、オープンソースのビルドステップが提供されている
- カスタム ビルドステップ
    - ビルドで使用する独自のビルドステップを作成できる

各ビルドステップは、ローカルの Docker ネットワーク `cloudbuild` に接続しているコンテナで実行され、ビルドステップが相互に通信を行い、データ（ファイル等）を共有できる。

### 2.1. ビルドのライフサイクル

一般的な Cloud Build ビルドのライフサイクルは以下の通り。

1. アプリケーション コードと必要なアセットを準備
2. Cloud Build の命令が含まれる YAML または JSON 形式のビルド構成ファイルを作成
3. Cloud Build にビルドを送信
4. Cloud Build は提供されたビルド構成に基づいてビルドを実行
5. 該当する場合、ビルドされたイメージは Container Registry に push される

ビルドを Cloud Build に送信する前にテストする場合に、`cloud-build-local` ツールを用いてローカルでビルドを実行できる。

### 2.2. ビルド構成ファイル

[ビルド構成](https://cloud.google.com/build/docs/build-config?hl=ja) ファイルは YAML または JSON 構文で記述する。  
curl などのサードパーティの http ツールを使用してビルド リクエストを送信する場合は、JSON 構文を使用する。

```yaml
steps:
- name: string
  args: [string, string, ...]
  env: [string, string, ...]
  dir: string
  id: string
  waitFor: [string, string, ...]
  entrypoint: string
  secretEnv: string
  volumes: object(Volume)
  timeout: string (Duration format)
- name: string
  ...
- name: string
  ...
timeout: string (Duration format)
queueTtl: string (Duration format)
logsBucket: string
options:
 env: [string, string, ...]
 secretEnv: string
 volumes: object(Volume)
 sourceProvenanceHash: enum(HashType)
 machineType: enum(MachineType)
 diskSizeGb: string (int64 format)
 substitutionOption: enum(SubstitutionOption)
 dynamicSubstitutions: boolean
 logStreamingOption: enum(LogStreamingOption)
 logging: enum(LoggingMode)
substitutions: map (key: string, value: string)
tags: [string, string, ...]
serviceAccount: string
secrets: object(Secret)
availableSecrets: object(Secrets)
artifacts: object (Artifacts)
images:
- [string, string, ...]
```

## 3. Cloud Tasks

[docs](https://cloud.google.com/tasks/docs/concepts?hl=ja)

Cloud Tasks と Cloud Pub/Sub は類似しているように思える。  
しかし、最大の違いは Cloud Tasks は Publisher が **明示的に** タスクをコントロールできることにある。  
Cloud Pub/Sub は、 Publisher が Subscriber をコントロールすることはできない。