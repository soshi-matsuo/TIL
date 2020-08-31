## gRPC
### 概要
Google製の、HTTP/2とProtocol Buffersを利用したRPCフレームワーク（現在はOSS）。  
Protocol Buffersで送受信データをシリアライズすること＆HTTP/2を利用することで高速なRPCを実現している。  
特にバックエンドのマイクロサービス間通信において、REST APIの代替としての注目株。  
### アーキテクチャ
gRPCは、一般的なWebサービス同様C/S型の構造を取る。  
- サーバサイドでは、後述する`proto`ファイルで定義された、サービスのメソッドの実際の処理を実装した後、gRPCサーバを実行してクライアントのリクエストを待機＆処理する。  
- クライアントサイドでは、Stubオブジェクト（サービスと同じメソッドを実装）に対するメソッド呼び出しを介して、サーバ上のサービスのメソッドを呼び出し、レスポンスを受け取る。  
  
C/S間でデータが送受信される際の実際の流れ（Lifecycle）は、以下のgRPCの4つの通信方式に依存し、どの通信方式かはサービスのメソッドシグネチャによって規定される  
- Unary RPC  
- Server streaming RPC  
- Client streaming RPC  
- Bidirectional streaming RPC  
### 依存・関連技術
**RPC：Remote Procedure Callの略で、リモートサーバ（ネットワーク越しの別のマシン）上のメソッドを実行するための規約。**  
- C/S型のサービス間通信の手法の1つで、技術自体はインターネット普及以前から存在する。  
- クライアントがサーバに対して呼びたい関数とそこに与える引数を指定すると、サーバからその実行結果が返る、というのが基本的な流れ。  
- クライアントアプリが自らのオブジェクトのメソッドを呼ぶのと同様にサーバ上のメソッドを実行できるので、分散型アプリケーション/サービスの実装に適している。  
- RPC APIとREST APIの最も大きな違いは、アドレス可能なものがプロシージャ（関数）なのかリソース（データ）なのかという点。  
- Webを介してRPCで通信する手段としては、HTTPベースでデータフォーマットとしてXML/JSONを利用するXML-RPC/JSON-RPCが存在した。  
  
**Protocol Buffers：Googleが開発したデータシリアライズ形式の1種。**  
シリアライズ：メモリ内の構造化データを、ネットワークで送信したりストレージに記録したりするために、「ひとまとまりの階層を持たないフラットなデータ構造」に復元可能な形で変換すること。  
  - 具体的には、シリアライズされたデータは文字列またはバイト列になる。  
  - デシリアライズ：文字列/バイナリから構造化データを復元すること。  
  
Protocol Buffersは、シリアライズ用のスキーマ定義を行うための**DSL（ドメイン固有言語）**でもある。  
- gRPCでは、この性質からIDL(Interface Definition Language)としても利用されており、`proto`ファイルに定義したAPIの仕様（サービス定義）を元にC/Sの雛形コードを生成できる。  
  - `proto`ファイルに、gRPCサービスとそのメソッド（関数）、メソッドが引数/返り値として利用するProtocol Buffers Message（ペイロード）の2つを定義する。  
  - `proto`ファイルをgRPC付属の`protoc`でコンパイルすると、多言語対応の雛形コードが生成される。  
  
**HTTP/2：TBW**  
- 参考  
  - https://qiita.com/mogamin3/items/7698ee3336c70a482843  
  - https://qiita.com/namusyaka/items/71cf27fd3242adbf348c  
  
**grpc-gateway：gRPCのサービスをREST APIとして実行できるようにするリバースプロキシを生成するプラグイン。**  
- `proto`ファイルから、リバースプロキシ、gRPCサービスのコードを自動生成する。  
- プロキシ：クライアントとサーバを仲介する役割を持つサーバ。  
  - フォワードプロキシ（プロキシ）：あるサーバへのリクエストをクライアントの代わりに送信するサーバのことで、目的のサーバへ直接アクセス出来ないクライアントがリクエストを送るために使われる。  
  - リバースプロキシ：特定のサーバへのアクセスをそのサーバの代理で受け取るサーバのことで、ロードバランシング等の目的で使われる。  
- C/RP間はREST APIの通信（基本的にはJSON over HTTP/1.1）になるので、HTTP/2による高速性の恩恵は受けられなくなる。  
  - 逆に言えば、クライアントから直接gRPCサービスにアクセスするにはgrpc-gatewayが必須。  
### 参考文献
- gRPC概要 in Qiita： https://qiita.com/gold-kou/items/a1cc2be6045723e242eb  
- スキーマ言語としてのProtobuf： https://qiita.com/yugui/items/160737021d25d761b353  
- gRPC公式（Introduction）： https://grpc.io/docs/what-is-grpc/introduction/  
- gRPC公式（Lifecycle）： https://grpc.io/docs/what-is-grpc/core-concepts/#rpc-life-cycle  

Nodeでの実装レベルについては、8月3週目のBoxを参照
