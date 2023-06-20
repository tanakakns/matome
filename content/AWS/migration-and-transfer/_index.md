---
title: "移行と転送"
date: 2023-05-30T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 16
---

- **AWS Schema Conversion Tool(AWS SCT)**
  - 異なるデータベースエンジン間で既存のデータベーススキーマを移行先のデータベーススキーマに変更することができるツール（サービスではない）
  - 変換のみを行うため、実際のデータ移行を行えるわけではない
  - 同じデータベースエンジン間でのデータ移行には **AWS DataMigration Service(AWS DMS)** を利用する
  - AWS SCTとAWS DMSを組み合わせることで、異なるデータベースエンジン間でのスキーマ変換とデータ移行を行うことができる
- **AWS Transfer Family**
  - AWSのストレージ（S3、EFS）にSFTP、FTPS、FTP、AS2 プロトコルを利用してファイル転送を可能にするサービス
- **AWS DataSync**
  - NFS、SMB、HDFS、Amazon S3、Amazon EFS、Amazon FSx間で一度のデータ移行を行う際に利用できる
- **AWS Storage Gateway**
  - オンプレミスのソフトウェアアプライアンスをクラウドベースのストレージと接続し、オンプレミスの IT 環境と AWS のストレージインフラストラクチャとの間にデータセキュリティ機能を備えたシームレスな統合を実現するサービス

<!--more-->

<!-- {{% children description="true"   %}} -->