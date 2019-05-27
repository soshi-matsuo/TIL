# Learn  

## HTTP
### Same Origin Policy(SOP)とCORS(Cross Origin Resource Sharing)
- オリジン：プロトコルスキーム＋ホスト＋ポート番号（通常省略）からなるURIのこと  
- SOP：**オリジンAで実行されているWebアプリは、別のオリジンBとリソースのやり取りをすることはできない**という、悪意あるWebサイト運営者からの攻撃・妨害を防ぐためのセキュリティ上の仕組み。同一生成元ポリシーとも。  
  - **通常のWebページは、インターネットを介して外部サーバーからリソースを読み込むためのリクエストを頻繁に送信する**が、この通信を無制限に許可しているとセキュリティリスクが有る。よってこのような操作は**ブラウザによって**制限される。  
- CORS：Webアプリによる、異なるオリジン上のリソースへのアクセス。またはそれをブラウザで管理する仕組み。  
  - 「どのオリジンがこのサーバー上のリソースにアクセスできるか」をリソースのサーバー側で指定して、それをブラウザに伝えることでアクセスを許す。  
  - 具体的には、アクセスを受けるサーバー側で、CORS用のHTTPヘッダー（許可するオリジンを指定）を追記したレスポンスを返すようにする。  
  - オリジンのみならず、そこから送られるべきHTTPメソッドもレスポンスヘッダーで指定できる。
- Preflightリクエスト：一部のCORSリクエストについて、サーバーに安全性を確かめるためのリクエスト  
  - CORSでは、以下のいずれかに該当する場合、元々のリクエストを送信する前に`OPTIONS`リクエストとしてPreflightを行うことが定められている  
    - HTTPリクエストのメソッドがGET, POST, HEAD以外である  
    - HTTPリクエストヘッダに`Accept`, `Accept-Language`, `Content-Language`以外のフィールドが含まれている。あるいは、`Content-Type`フィールドに`application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`以外の内容が指定されている
![SOPとCORS](https://mdn.mozillademos.org/files/14295/CORS_principle.png)

## プログラミング
- Stateパターン：デザインパターンの一種。あるクラスの振る舞いが状態に応じて変化する、というような実装をするとき、**そのクラスの状態自体をクラスとして切り分けて入れ替え可能にする**手法。[参考](https://www.techscore.com/tech/DesignPattern/State.html/)
### Java
- Tomcat：Java Servlet（Javaで作成された、サーバー上でWebページの動的生成やデータ処理を行うためのプログラム）を動かすためのサーブレットコンテナ。簡易的なWebサーバ機能も持つ。  
- サーブレットフィルタ：サーブレットコンテナとサーブレットがリクエストとレスポンスをやり取りする間に介入し、サーバー上でリクエストを前処理orレスポンスを後処理するプログラム  
  - JavaのWebアプリはクライアント↔サーブレットコンテナ↔（フィルター）↔サーブレットという構造になっている  
- JerseyとSpring bootの関係→http://zetcode.com/springboot/jersey/　　
  - JerseyはRESTなWebアプリを開発するためのJAX-RSの参照実装で、`spring-boot-starter-jersey`はJerseyをSpring bootで扱うためのDependency。`spring-boot-starter-web`はSpring MVCを扱うが、前者は後者を代用できる
- Antのパターン書式：Antでビルドする際に、条件に合致するファイルやディレクトリを設定ファイルで指定するための書式が元。正規表現に似ているが別物  
  - 「?」：任意の一文字→regexの「.」  
  - 「\*」：0個以上の任意の一文字→regexの「.\*」
  - 「/」：一個の任意のディレクトリ  
  - 「\*\*」：0個以上の任意のディレクトリ。パスの最後に指定された場合は、そこの階層以下の全ディレクトリとファイルを指す
## DevOps
- 要するに、開発と運用の密接な連携によってスピードとクオリティを向上する、というコンセプト  
  - Agile, CI/CD, Microservices, Containerなどの開発手法や技術と関係が深い  
    - CI/CD：Continuous Integration(プッシュと同時に自動テスト), Continuous Deliver(CI+ステージング), Continuous Deployment(CD+マスターデプロイ）。Jenkinsがツールとして有名  
    - Microservices：ある大きなサービスの開発において、用途別の小さいサービスを組み合わせて開発するというアーキテクチャ。システムを複数のより小さなシステムの集合体とみなす。  
    - [Container](https://employment.en-japan.com/engineerhub/entry/2019/02/05/103000)：ホストOS上の、独立隔離されたアプリケーション実行環境。ホストOSとは別のプロセス管理を行うが、コンテナ自体はホストOS上のプロセスの一つとして管理される。  
      - 複数のコンテナが、Kubernetesなどのオーケストレーションツールによって連携して管理される
