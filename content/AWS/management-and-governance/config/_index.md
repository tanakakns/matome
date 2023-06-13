---
title: "AWS Config"
date: 2023-06-09T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 2
---

<!--more-->

[AWS Config](https://docs.aws.amazon.com/ja_jp/config/latest/developerguide/WhatIsConfig.html)についてまとめる。

- 1. [概要](#1-概要)

## 1. 概要

AWS Configとは、AWSアカウントにあるAWSリソースの設定を評価、監査、審査できるサービス。

- AWSリソースの変更履歴やリソース間関連性の追跡管理
  - CloudTrailと統合することで、変更とイベントの関連付けが可能
- AWSリソースの状態評価
  - ルールを設定することでAWSリソースの不適切な設定を検出しSNSなどで通知が可能
- ダッシュボードがあり、全体的なコンプライアンス状況を確認できる
- AWS Config はマルチアカウント・マルチリージョンに対応

なお、よく似たサービスに **AWS Trusted Advisor** があるが、これはサービスの使い方/設定に対して「AWS Well-Architected Frameworkに準じているか」を確認して、NGポイントを教えてくれるサービス。