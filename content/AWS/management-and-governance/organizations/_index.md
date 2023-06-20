---
title: "AWS Organizations"
date: 2023-06-09T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

<!--more-->

[AWS Organizations](https://docs.aws.amazon.com/ja_jp/organizations/latest/userguide/orgs_introduction.html)についてまとめる。

- 1. [概要](#1-概要)
- 2. [仕組み](#2-仕組み)

## 1. 概要

- AWS Organizationsは、複数のAWSアカウントを一元管理できるサービス
- 複数のAWSアカウントをグループ化し、そのグループ毎に共通のポリシーを設定できる

## 2. 仕組み

- AWS Organizations には以下の機能がある
  - AWSアカウントの一元管理
  - 新規アカウントの作成・管理
  - 請求とコストの一元管理
  - アカウント間のリソース共有
- 複数のAWSアカウントを **組織** として管理する
- 組織には 1 つの **管理アカウント** と 0 以上の **メンバーアカウント** が所属する
  - 管理アカウントはメンバアカウントを作成できる
  - 別の組織に所属するメンバーアカウントを自分の組織に招待したり、自組織からメンバーアカウントを離脱させることも可能
  - なお、管理アカウントでは AWS CloudTrail を例外として AWS リソースを作成しないことをおすすめする（メンバーアカウントでやる）
- 管理アカウントは組織全体の管理権限を持ち、すべてのメンバーアカウントの **一括請求** に対応する
- 組織内では、複数のメンバーアカウントを **組織単位**（ **OU** ： Organization Unit）としてグループ化できる
- OUごとに **サービスコントロールポリシー**（ **SCP** ）を設定でき、また、管理アカウントでは組織全体にSCPを適用できる
  - SCPで設定された権限は、いかにそのAWSアカウントのIAMにて強い権限を設定したとしてもSCPの権限設定が優先される
  - メンバアカウントのルートユーザでさえも SCP により制限される
- AWS Resource Access Manager( **AWS RAM** ) を用いて、AWSアカウント間やOU間でリソースの共有を行える

参考：https://www.ashisuto.co.jp/db_blog/article/aws-organizations.html

## 3. SCPを使用した戦略

- 拒否リスト戦略
  - アクションはデフォルトで許可され、禁止するサービスとアクションを指定する
- 許可リスト戦略
  - アクションはデフォルトで禁止され、許可するサービスとアクションを指定する