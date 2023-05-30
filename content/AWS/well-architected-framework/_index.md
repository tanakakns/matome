---
title: "AWS Well-Architected フレームワーク"
date: 2021-01-30T18:49:21+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 23
---

<!--more-->

- [AWS Well-Architected フレームワーク：ホワイトペーパーHTML](https://wa.aws.amazon.com/index.ja.html)
- [AWS Well-Architected フレームワーク：ドキュメント](https://docs.aws.amazon.com/ja_jp/wellarchitected/latest/framework/welcome.html)
- [AWS Well-Architected Tool](https://aws.amazon.com/jp/well-architected-tool)

## 5 つの設計原則

5 つの柱とも。

1. [運用上の優秀性](https://d1.awsstatic.com/whitepapers/ja_JP/architecture/AWS-Operational-Excellence-Pillar.pdf)
    - 開発をサポートし、ワークロードを効率的に実行し、運用に関する洞察を得て、ビジネス価値をもたらすためのサポートプロセスと手順を継続的に改善する能力。
2. [セキュリティ](https://d1.awsstatic.com/whitepapers/ja_JP/architecture/AWS-Security-Pillar.pdf)
    - このセキュリティの柱では、データ、システム、資産を保護して、クラウドテクノロジーを活用し、セキュリティを向上させる機能。
3. [信頼性](https://d1.awsstatic.com/whitepapers/ja_JP/architecture/AWS-Reliability-Pillar.pdf)
    - 期待される時に意図された機能を正しく一貫的に実行するワークロードの能力。
    - これには、ワークロードのライフサイクル全体を通じてワークロードを運用およびテストする能力が含まれる。
    - このホワイトペーパーでは、AWS で信頼性の高いワークロードを実装するための詳しいベストプラクティスガイダンスを提供する。
4. [パフォーマンス効率](https://d1.awsstatic.com/whitepapers/ja_JP/architecture/AWS-Performance-Efficiency-Pillar.pdf)
    - システムの要件を満たすためにコンピューティングリソースを効率的に使用し、要求の変化とテクノロジーの進化に対してもその効率性を維持する能力。
5. [コスト最適化](https://d1.awsstatic.com/whitepapers/ja_JP/architecture/AWS-Cost-Optimization-Pillar.pdf)
    - 最も低い価格でシステムを運用してビジネス価値を実現する能力。

## 1. 運用上の優位性

### 1.1. 設計原則

1. 運用をコードとして実行する
    - クラウドでは、アプリケーションコードに使用しているものと同じエンジニアリング原理を環境全体に適用できます。
    - ワークロード全体 (アプリケーション、インフラストラクチャ) をコードとして定義し、コードを使用して更新できます。
    - 運用手順をコードとして実装し、イベントに対応してそのコードをトリガーすることで自動的に実行できます。
    - 運用をコードとして実行することで、人為的なミスを抑制し、イベントへの一貫性のある対応を実現できます。
2. 小規模かつ可逆的な変更を頻繁に行う
    - コンポーネントを定期的に更新できるようにワークロードを設計します。
    - 変更は、失敗した場合に元に戻すことができるように小規模に行います (可能な場合は、顧客に影響がないようにします)。
3. 運用手順を定期的に改善する
    - 運用手順を実施するときに、改善の機会を探します。
    - ワークロードを改良するときに、手順もそれに応じて改良します。
    - 定期的なゲームデーを計画し、すべての手順が効果的で、チームがその手順を熟知していることを確認および検証します。
4. 障害を予想する
    - 障害の考えられる原因を除去または軽減できるように、原因を特定する「プレモータム」演習を実施します。
    - 障害シナリオをテストし、その影響に関する理解を検証します。
    - 対応手順をテストし、手順が効果的で、チームが手順の実行を十分に理解していることを確認します。
    - 定期的なゲームデーを計画し、ワークロードと、シミュレートされたイベントに対するチームの応答をテストします。
5. 運用上のすべての障害から学ぶ
    - 運用上のすべてのイベントと障害から教訓を学び、改善を促進します。
    - チーム間と組織全体で教訓を共有します。

### 1.2. ベストプラクティス

1. 組織
2. 準備
3. 運用
4. 進化