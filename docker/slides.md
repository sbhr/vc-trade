# Docker

参考：[プログラマのためのDockerの教科書](https://www.shoeisha.co.jp/book/detail/9784798144627) 筆者が考える良いアプリケーション実行環境

- シンプルで簡単なインフラ構成にする
- だれにでも理解できる状態を持つ
- 繰り返し行う作業はコンピュータに任せる

# おさえておきたいシステム／インフラの知識

Docker：コンテナ仮想技術を使ったアプリケーション実行環境を作成／管理するツール

## システム基盤

アプリケーションを稼働させるために必要になるハードウェアやOS／ミドルウェアなどのインフラのこと
多くの企業で採用されている利用形態は主に3つ

- オンプレミス（on-premises）
  - 自社でデータセンターを保有して、システム構築から運用までを行う。ハードウェアやOS、ネットワーク機器等を全て自社で調達し、管理を行う。多くの企業で採用されてきた。 初期コストが大きく、運用にかかる費用もシステムの利用量にかかわらず一定量必要となる。
- パブリッククラウド（public cloud）
  - インターネットを介して不特定多数に提供されるクラウドサービス。サーバやネットワークなどのインフラに関する初期コストが不要。IaaS, PaaS, SaaSなど。システム基盤でIaaS、利用した時間やデータ量に応じて料金を払う。
- プライベートクラウド（private cloud）
  - 特定の企業グループにのみ提供されるクラウドサービス。企業内でデータセンターを保有するイメージ。セキュリティが担保しやすかったり、独自の機能やサービスを追加しやすかったり。

### クラウドが適しているケース

- トラフィックが変動しやすいシステム
- 災害対策で日本以外にバックアップを構築したいシステム
- なるべく早く稼働させたいシステム

### オンプレミスが適しているケース

- 高い可用性が求められるシステム
- 機密性の高いデータを扱うシステム
- 特殊な要件があるシステム
- トータルコストが高くなるシステム
  - ユーザごとに課金やデータ量に応じて課金など

## ちょっとまとめ
どのシステムをオンプレミスで残し、どのシステムをクラウドに移行するか、また新システムの導入時など、どちらが適しているか見極めることが重要
**アプリケーションの移植性とインフラを含むシステムの構成管理のしやすさを考えること** が大切

### システム基盤の構築／運用の流れ
システム化計画／要件定義  
↓  
インフラ設計  
↓  
インフラ構築  
↓  
**運用**

システムを安定稼働させるためには運用設計が重要。
保守工数を減らすためには、システム運用で自動化できるところを可能な限り自動化するよう設計する必要がある。
#### 自動化のメリット
- オペレーションミスがなくなる
- 属人化しやすい運用業務を可視化できる
- 自動化により空いた工数で、予防保守や新技術の調査／導入など

**Dockerはシステム構築やシステム運用で、これまで人の手によって行われてきた作業のを多くを自動化するためのプラットフォーム**