# Spring Cloud
Spring bootアプリケーションにおいて、分散システム構築に必要な各種パターンを簡単に実装するためのライブラリで、クラウドネイティブアプリケーション実装を助けるツールの集合体とも。  
## Bootstrap Context
Spring Cloudアプリが動作するために必要なもので、アプリが利用する外部設定サーバへの接続や、ローカルの設定値の暗号化/復号を行う。  
Application Contextによってアプリの設定値がマッピングされるのに先立って処理される。  
クラウドネイティブでは、実際にアプリ内で利用する設定値をリモートのConfiguration Server上に持つ設計を採ることが多く、実際にアプリをセットアップする前にそこから情報を取得する必要があり、これがそのままBootstrap ContextとApplication Contextの役割分担になっている。  
BootstrapContextを構成するには`bootstrap.yml`がよく使われ、最低限以下の二つを記述する。  
- 外部のConfiguration Serverの接続情報  
- Configuration Server上でのアプリの名前（正しい設定値を取得するためのID）
## `@RefreshScope`
### 概要
Configuration Serverの設定値の更新を、Spring Cloudアプリを動かしたまま反映するためのアノテーション。  
これを付与したBeanには、publicメソッドとして`refreshAll()`, `refresh(<String>)`が定義される。このメソッドはアプリ（Config Clientと呼ばれる）の`/refresh`エンドポイントにPOSTを送ることで実行され、**新しい設定値を利用してそのBeanを再生成する**。  
実際にBeanが作り直されるのは、**設定値更新後にそのBeanのメソッドを初めて呼び出したタイミング**（遅延評価）である点に注意。  
### Tips
- アプリに上述の`/refresh`エンドポイントを設けるにはspring-boot-starter-actuatorが依存関係に必要  
- **`@ConfigurationProperties`はデフォルトで`@RfreshScope`が適用されている**
- このアノテーションを使わずにBeanを更新するには、`/restart`にアクセスしてDIコンテナを再起動する必要がある
## Spring Cloud Consul
Spring bootアプリへのConsul導入を補助するSpring Cloud配下のライブラリ。部署ではConsulのdistributed configuration(すべてのサービスインスタンスに同一の設定を供給する)機能を利用している。  
### Consul
一言でいうと、複数のサーバをオーケストレーションするWebシステムメンテナンスツールで、クラウドネイティブなアプリの実現に貢献する。  
- オーケストレーション：複雑なシステム/ミドルウェア/サービスの、配備/設定/管理の自動化を指す用語（by Wikipedia）   
- 部署では、k8sでデプロイされた（=複数のサーバにまたがった？）アプリケーションに共通の設定を提供するConfiguration Serverとして利用されている。  
  - Consul上で設定値の更新があれば、連携しているアプリケーションにRefresh Eventを発行して更新を促すことができる。    
### distributed configurationの導入
#### Spring boot側の設定
- maven/gradleでspring-cloud-starter-consul-all/configを依存関係に追加  
- bootstrap.ymlのspring.cloud.consulに、以下のプロパティを設定する  
  - host, portにConsulサーバの接続情報を定義  
  - config.enabledをtrueに定義してConsul Configを有効化  
#### Consul側の設定
- Consul上では`config/<spring.application.name>`以下に設定値を格納しておく  
  - アプリ側で`spring.profile.active=<env>`が設定されている場合は`config/myApp,staging`のようなパスになる  
  - 設定値のディレクトリ構造は、アプリ側のプロパティの階層構造と対応する。つまり、my.propというプロパティの値をConsulに格納したい場合は `config/myApp/my/prop`に定義する  
- `bootstrap.yml`で`config.format`をYAML/PROPERTIESに定義すれば、Consul上でもyml/propertiesファイルのような形でデータを保持できる  
  - YAMLに設定した場合、`config/<name>/data`以下に値を持たなければならない  
  - Consul CLIから`consul kv put config/<name>/data @<filename>.yml`で、dataにファイルをアップできる
### 設定値の更新
Consulを利用するならば、上述の基本設定に加えて以下を`bootstrap.yml`で定義すれば、アプリ上での設定値更新の監視がより簡単になる  
- `config.prefix`で監視したい設定値のベースディレクトリを定義する  
- `config.watch.enabled`をtrueに定義する（Config Watchを有効化）  

これにより、prefix以下の設定値が変更された時、ConsulからSpringアプリへ自動的にRefresh Eventが発行されるが、これは`/refresh`へのPOSTと同様に扱われる。  
つまり、ConfigWatchを有効化し、Beanに`@RefreshScope`を設定しておくだけで、**Consul上の設定変更をSpringアプリで自動的に反映することができる**。  
### 参考
- https://www.baeldung.com/spring-cloud-consul  
- https://cloud.spring.io/spring-cloud-static/spring-cloud-consul/2.2.2.RELEASE/reference/html/#spring-cloud-consul-config  
