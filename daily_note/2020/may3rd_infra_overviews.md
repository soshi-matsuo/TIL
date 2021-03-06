# インフラ関連用語、技術の概要について
[リンク](https://www.atmarkit.co.jp/ait/articles/1901/29/news005.html)の通り、システムアーキテクチャのトレンドは専用コンピュータ→汎用物理サーバ→仮想サーバ→クラウド→コンテナと変遷してきた。  
この流れの裏には、システムコストの最小化＆サービス提供の加速というモチベーションが存在する。
## 仮想サーバ
### 概要
仮想サーバとは、物理的なハードウェア内に構築された「サーバのように振る舞うもの」のことで、物理的なハードウェアのリソースを分割して割り当てることで実現される。  
仮想化とは、**物理的なハードウェアのリソースを論理的に分割/統合する技術**を指し、後述のクラウドサービスの要素技術となっている。  
仮想化を実行して仮想サーバを構築するための環境をハイパーバイザとも呼ぶ。  
仮想サーバはVirtual Machine(VM)と呼ばれることも多く、VMの各コンポーネントをvCPUのようにv〇〇と呼ぶ。  
### メリット
- 複数OSの利用が可能  
  - 仮想サーバ上にインストールしたOSをゲストOSと呼び、複数の仮想サーバによって、1つの物理サーバの上で複数のOSを利用することができる  
  - 従来は個別にHWが必要だったのが1台の高性能サーバで済むので、コスト削減になる  
- 権限の切り分けが楽  
  - 通常のOSの権限分配は、何でもできる管理者vs制限のキツイ一般ユーザの構造なので、サービスごとに仮想サーバを立てて、各担当者に管理者権限を割り振れば簡単  
- リソースの切り分けが可能  
  - 仮想サーバごとにメモリ容量やCPU数を簡単に指定可能←UNIX系は標準の方法がないらしく、仮想化で代替されている  
- 運用管理の自動化が可能  
  - ソフトウェアによってサーバの起動、停止、構成変更などが可能なので自動化しやすい  
  - 仮想サーバのコピーが作れるので、同じ環境を簡単に再現できる  
### デメリット
- 少しの待ち時間の増加でも問題になる場合には向かない  
- ハイパーバイザが対応していないマイナーなハードウェアの機能は利用できない  
## クラウド
ネットワークを通して種々のITリソースを提供するようなサービスのことをクラウドサービスと呼ぶ。  
ベンダーは、自社のリソースを仮想化技術で統合/分割することで、複数のユーザへの柔軟なリソース提供を実現している。  
対義語としてオンプレミスという言葉が多用され、これはインフラ用のハードウェアを組織の自前で保有、運用するシステム形態を表す。  
オンデマンドで必要なだけのリソースを利用可能で、従量課金制がメジャーなので、システムの初期投資やセキュリティなどのコストを削減できる点で人気？  
### 主なサービス形態
- IaaS：サーバ、ネットワーク機能、ストレージ等のインフラリソースを提供するサービス  
  - ex. VMとしてEC2、GCEなど、ストレージとしてS3、GCSなど  
- PaaS：上記に加えてミドルウェア（OS、開発用フレームワーク、実行環境）を提供する  
  - 要するに、アプリを動かすための環境をサービスとして提供している？  
  - ex. GAE, Azure, Herokuなど  
- SaaS：プロバイダ上で稼働し、インターネットを介して提供されるアプリケーションを指す  
  - ex. Gmailなど  
### クラウドネイティブ
クラウドの利点をフル活用してアプリ・サービスを開発・提供するためのアプローチのことで、Linux Foundationを母体に持つCloud Native Computing Foundationが提唱している。具体的には、クラウド環境で拡張性と開発・提供速度を向上するような技術・手法がクラウドネイティブであるとされる。  
抽象度が高いが、以下のような技術や概念が要素とされることが多い。  
- Immutable Infrastructure：アプリや依存ソフトウェアに変更がある度に、それを反映した新しい環境をデプロイしてリクエストの行き先を切り替え、古い環境を破棄するという手法  
- コンテナ化：後述  
- マイクロサービス：独立したモジュールに分割された複数サービスが、独立なプロセスとしてネットワーク上に存在し、APIを通じて疎に連携して動作することで1つのアプリケーションとなる、という設計  
- サービスメッシュ：マイクロサービス上のサービス間通信を管理するための技術で、各サービス間の通信を中継するサイドカープロキシとその管理コンポーネントを併せ持つソフトを指す？  
  - 主要なものとしてIstio、Consulなど  
  - 以下のマイクロサービス特有の要求を解決するために導入された  
    - Service Discovery：通信したいサービスのネットワーク上の位置（接続情報）を特定する仕組みで、DNSはこの一種。クラウド環境ではIPアドレスやホスト名が動的に変化する可能性があり、それを踏まえてService Discoveryを行うのが難しい  
    - 障害の分離：あるサービスに生じた障害によってアプリ全体にまたがる大規模障害が生じる可能性があり、それを防ぐためのサーキットブレーカーが必要  
    - 分散トレーシング：各サービス間のreq, resを監視したい  
    - 認証・認可：各サービスの権限に応じて提供するコンテンツを制御したい  

