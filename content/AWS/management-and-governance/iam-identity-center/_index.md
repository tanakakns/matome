---
title: "AWS IAM Identity Center"
date: 2023-06-09T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 4
---

<!--more-->

[AWS IAM Identity Center](https://docs.aws.amazon.com/ja_jp/singlesignon/latest/userguide/what-is.html)についてまとめる。

- 1. [概要](#1-概要)

## 1. 概要

旧AWS SSO。

- 複数AWSアカウントのIAMを統一的に管理することができるサービス
- IAM Identity Centerは既存のIAMユーザーと競合しないので、段階的に一部のユーザーだけ試してみるといった運用も可能
  - 同じユーザー名でも問題ない
- 外部のIdPとも連携可能