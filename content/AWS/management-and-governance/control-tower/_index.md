---
title: "AWS Control Tower"
date: 2023-06-09T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 3
---

<!--more-->

[AWS Control Tower](https://docs.aws.amazon.com/ja_jp/controltower/latest/userguide/what-is-control-tower.html)についてまとめる。

- 1. [概要](#1-概要)
- 2. [用語](#2-用語)

## 1. 概要

- **AWS Control Tower** は、セキュアなマルチアカウント環境を自動でセットアップするサービス
- AWS Organizations、AWS Service Catalog、AWS IAM Identity Centerなど、他のいくつかの AWS のサービスの機能の「オーケストレーション」を行い、1 時間足らずでランディングゾーンを構築
- マルチアカウントの一元管理自体は AWS Organizations で実現できるが、アカウントの設計やどのようなポリシーを設定するかは自分たちで検討・実装しなければならない
- AWS Control Tower は AWS Organizations をベースとした環境を、AWSのベストプラクティスに則った形で自動セットアップできる

組織の管理アカウントにて実行可能。

## 2. 用語

- **ガードレール**
  - AWS利用者がセキュリティ上問題のある操作をしないよう検知・防止する仕組み
  - 利用者の手を止めることなく、セキュアに利用できる状態を目指す
  - AWS Control Towerでは「コントロール」と呼ばれる実装がガードレールに当たる
- **ランディングゾーン**
  - よいマルチアカウント統制環境の総称・考え方
  - 安全にアカウント・ワークロードを追加・開始できる環境を構築するための設計、仕組みなどをまとめたもの
- ダッシュボード
  - AWS Control Towerでは、作成した環境を効率的に管理するためのダッシュボードが用意されている
  - 作成したグループやアカウントの数、ガードレールの数、検出された違反の数などがまとめて表示される