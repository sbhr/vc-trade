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

# コンテナ仮想化技術と Docker
Docker はコンテナ仮想化技術を使ったアプリケーションの実行環境を構築、運用するためのプラットフォーム
## 仮想化技術
物理的に本番環境と同じ環境を整えるのはとても大変（金額、管理…etc）

代表的な仮想化技術
- ホスト型仮想化
- ハイパーバイザー型仮想化
- コンテナ型仮想化


### ホスト型仮想化
ハードウェアの上にベースとなるホストOSをインストールし、ホストOSに仮想化ソフトウェアをインストールする。
仮想化ソフトウェア上で、ゲストOSを動作させる。
ホストOSの上でゲストOSを動かすのでオーバーヘッドが高くなる。

仮想化ソフトウェア
- [Oracle VM VirtualBox](https://www.virtualbox.org)
- [VMWare Player](https://www.vmware.com/jp)

### ハイパーバイザー型仮想化
ハードウェア上に仮想化を専門に行うソフトウェアである「ハイパーバイザー」を配置し、ハードウェアと仮想環境を制御する。

```
[仮想環境] … [仮想環境]
[  ハイパーバイザー   ]
[   ハードウェア     ]
```

代表的なソフトウェア
- [Hyper-V](http://www.microsoft.com/ja-jp/server-cloud/products-windows-server-2012-r2.aspx)
- [XenServer](http://www.citrix.co.jp/products/xenserver/overview.html)

### コンテナ型仮想化
OSやハイパーバイザーの上でさらにOSを複数動かすとどうしても多くのリソースを必要とする。
そこで、ホストOS上に論理的な区画（**コンテナ**）を作り、アプリケーションを動作させるのに必要なライブラリやアプリケーションなどをコンテナ内に閉じ込め、
あたかも個別のサーバの用に使うことができるようにしたものが、コンテナ型仮想化。
- ホストOSのリソースを論理的に分割し、複数のコンテナで共有
- オーバーヘッドが少ないため、軽量で高速に動作

```
[コンテナ] … [コンテナ]
[  ハイパーバイザー   ]
[   ハードウェア     ]
```

## コンテナ仮想化技術の歴史
### FreeBSD Jail (2000年)
管理者権限のスコープが Jail 内に制限されている。
そのため、一般ユーザに管理者権限を与えることができる。(shutdown, reboot等)
- プロセスの区画化
  - 同じ Jail で動作するプロセスにしかアクセスができない。
- ネットワークの区画化
  - Jail には IPアドレスが割り当てられ、外にアクセスするにはネットワーク経由でなければいけない。
- ファイルシステムの区画化
  - 操作できるコマンドやファイルを制限したい。

### Solaris Containers (2005年)
アプリケーションの実行環境をゾーンで区画化し、それらの区画を制御することで物理サーバを仮想化する。
Docker に似ている。
- Solaris ゾーン機能
  - ソフトウェアパーティショニング機能（1つのOS空間を仮想的に分割する）
- Solaris リソースマネージャ機能
  - 非大域ゾーンでCPUやメモリなどのハードウェアリソースを分配する

## Docker の特徴
コンテナ仮想化環境でアプリケーションを管理、実行するためのオープンソースプラットフォーム
Go言語で開発、2013 ~
さまざまな Linux ディストリビューション、クラウド環境で動作する。
Mac や Windows では Docker Toolboxというツールが提供されている。

### 移植性
システム開発において考慮しなければいけない要素

- アプリケーション本体
- ミドルウェアやライブラリ群
- OS/ネットワークなどのインフラ環境設定

Docker ではこれらのインフラ環境を**コンテナ**として管理する。
アプリケーションの実行に必要なすべてのファイルやディレクトリ群をまるごとコンテナとしてまとめる。
作成したコンテナのもとになるDocker イメージは DockerHub で共有する。

一度作ってしまえばどこでも動くソフトウェアの特性を**移植性（ポータビリティ）**と呼ぶ。
Dicker は移植性が高い。

### 相互接続性
さまざまな組織やシステムと連携して使うことができるソフトウェア特性を相互接続性（インターオペラビリティ）

- Red Hat Enterprise Linux 7（標準搭載）
  - Atomic Host（特化したホスト）
- Amazon Web Services
  - EC2 Container Service
- Jenkins
- GitHub
- Kubernetes

### Docker 専用 Linux ディストリビューション
軽量、高速を目的に不要なアプリケーションやコマンドなどが含まれていない
- Red Hat Enterprise Linux Atomic Host
- Project Atomic
- Snappy Ubuntu Core
- Core OS

コンテナ技術の標準化を目的とした Open Container Project なるものがあるらしい。

## Docker の基本機能
- Docker イメージを作る機能
- Docker コンテナを動かす機能
- Docker イメージを公開/共有する機能

### Docker イメージを作る機能
Docker イメージとは…アプリケーションの実行に必要になるプログラム本体/ライブラリ/ミドルウェア/OSやネットワークの設定などをまとめたもの
Dockerfile を元につくったり、既存のコンテナを更新したりすることで作成できる。
また、複数のイメージを重ねて別の新しいイメージを作成することができる。

### Docker コンテナを動かす機能
Docker イメージをもとにコンテナを動作させる。
コンテナ内で動作するプロセスを１つのグループとして管理し、グループごとにそれぞれファイルシステムやホスト名/ネットワークなどを割り当てる。
Linux のカーネル機能（namespace, cgroupsなど）を使っている。

### Docker イメージを公開/共有する機能
Docker イメージは Docker レジストリで一元管理できる。
[Docker Hub](https://hub.docker.com/) は公式の Docker レジストリ
Docker コマンドで、レジストリ上のイメージの検索アップロード、ダウンロードができる。
GitHub 上の Dockerfile を使って自動で Docker Hub にイメージをあげるといったこともできる。
Docker イメージは公開鍵暗号の仕組みを使った改ざん防止機能がある。

### Docker コンポーネント
コア機能を提供する Docker Engine を中心に、イメージを作成、公開、コンテナ実行をするためのさまざまなコンポーネントが提供されている。
- Docker Engine
  - Docker イメージの生成やコンテナの起動などを行う。
- Docker Kitematic
  - GUIツール
- Docker Registry
  - イメージを公開/共有するためのディレクトリ
- Docker Compose
  - 複数のコンテナ情報をコードで定義して一元管理
- Docker Machine
  - クラウド環境などに Docker の実行環境をコマンドで自動生成するためのツール
- Docker Swarm
  - 複数の Docker ホストをクラスタ化するためのツール

## Docker が動く仕組み
### コンテナを区画化する仕組み (namespace)
Docker はコンテナを区画化するために、namespace という機能を使っている。
プロセスに対して以下の６種類のリソースを独立した環境を構築できる。
- PID
- Network
  - IP、ポート番号、フィルタリング…
- UID
  - 例えば管理者権限のアカウントを名前空間外では一般アカウントとできる
- MOUNT
  - 隔離されたファイルシステムツリーを作る
- UTS
  - ホスト名やドメイン名を独自に持つ
- IPC（Inter Process Communication）
  - プロセス間通信

参考：[Docker内部で利用されているLinuxカーネルの機能 (namespace/cgroups)](https://qiita.com/wellflat/items/7d62f2a63e9fcddb31cc)

### リソース管理の仕組み (cgroup)
物理マシン上のリソースを複数コンテナに割り当てるために、cgroup を使っている。
Linux ではプログラムはプロセスとして実行され、このプロセスは1つ以上のスレッドのかたまりとして動作する。
cgroup はプロセス/スレッドをグループ化して、そのグループ内のプロセス/スレッドに対して管理を行う。
また、階層構造でプロセスをグループ化して管理できる。

cgroup で管理できるもの
- cpu
- memory
- devices
- freezer (グループに属するプロセスの停止/再開)
- net_cls (ネットワーク制御のタグを付加)

参考：[Docker内部で利用されているLinuxカーネルの機能 (namespace/cgroups)](https://qiita.com/wellflat/items/7d62f2a63e9fcddb31cc)

### ネットワーク構成（仮想ブリッジ/仮想NIC）
コンテナはサーバの物理NICとは別にコンテナごとに仮想NICが割り当てられる。
仮想NICは docker0 という仮想ブリッジに接続されコンテナ同士が通信する。
コンテナ同士で通信するためにはリンク機能が使われる。
リンク機能…コンテナにつけたエイリアス名が/etc/hosts等に反映されるため、名前を指定して通信できる。
（同一のホスト上に接続されたコンテナ同士でのみ有効）

外部ネットワークとの通信はNAPTで行われる。
