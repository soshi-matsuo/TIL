# Learn

## Security
OAuth：予め信頼関係を構築したサービスの間で、ユーザーの同意のもと、セキュアにユーザーの権限を受け渡しする、認可情報のサービス間委譲の仕組み  
- ユーザーは認可の際、認可情報が委譲される先のサービスにパスワードを教える必要はない。例えば、QiitaがGithub認証を提案してくる際、ユーザーがCredentialを入力するのはGithubのサイト上であり、QiitaはGithubからユーザーの認証が正しく行われたという証明（認可コード）だけを返す  
- つまり、あるサービスが別のサービスにリソースを扱う代理人としての許可を与える仕組み？  
- 参考
  - https://qiita.com/TakahikoKawasaki/items/e37caf50776e00e733be  
  - https://qiita.com/kojisaiki/items/48adf59d5d634fd330af  
  - https://murashun.jp/blog/20150920-01.html  
  - OAuth2.0vs1.0:https://stackoverflow.com/questions/4113934/how-is-oauth-2-different-from-oauth-1
  
## Database
MongoDB：Document型のNoSQLデータベースで、DB→Collection→Documentという構造になっている。RDBで言うレコードがCollection  
  - RDBがテーブルのスキーマ定義（フィールドに対する条件付など）を要する一方で、MongoDBのCollection内では、各ドキュメントが同じスキーマに基づいていなくてもよい  
    - 同じフィールドを持つ必要はないし、フィールド名が同じでもデータ型が異なることもある  
    - 実用上は、Schema Validationを使ってCollectionの更新やデータ挿入に対して一定の規則を設けることもできる
  - Documentは基本的にJSON形式で、Key:Valueで属性とその値を記述する  
    - あるDocumentの要素として別のDocumentを埋め込んでデータの関係を表すことができる（非正規化データモデル）  
    - 一方、RDBの外部キー参照のように、他のDocumentへの参照を格納することで関係を記述することもできる（正規化データモデル）
    
## Programming
- 宣言的（Declarative）：「どのように処理を行うのか」ではなく「何を行ってほしいか」だけを記述するプログラミングパラダイム。処理の内容はコンパイラに任される 
- エントリーポイント：プログラムやサブルーチンの実行を開始する場所  
- エンドポイント：APIにアクセスするためのURL
### Spring
RESTful関係のアノテーション  
- `@RestController`：HTTPリクエストのエントリーポイントとなるクラスを明記するための`@Controller` と `@ResponseBody`を組み合わせたアノテーション。これが付記されたクラス配下のリクエスト処理メソッド全てに`@ResponseBody`の効果が適用される。また、`@Controller`とは違ってJSONやXMLを返させるWebAPIにおいて使われる  
  - `@Controller`：`@Component`クラス(classpathスキャンの際にそのクラスをBeanとして自動検知させる)の特殊系。通常は`@RequestMapping`アノテーションと併用され、HTTPリクエストを扱うメソッドに付記される。また、主にWebページの開発に使われるため、戻り値はViewを指定するために使われる  
  - `@RequestMapping`：特定のURLへのHTTPリクエストとそれを処理するクラスを対応付けるためのアノテーション。具体的に「どのURLへの、どのHTTPメソッドを、どのメソッドが処理するか」は、`@GetMapping`や `@PostMapping`などをメソッドに付記することで明示する
  - `@ResponseBody`：リクエスト処理メソッドのシグネチャに付記することで、そのメソッドの戻り値が自動的にJSONにシリアライズされ、Javaのオブジェクト→HttpResponseと変換されて返される
- `@RequestBody`：送信されたHttpRequestを、このアノテーションを付記したJavaのクラスに自動でデシリアライズする。デフォルトでは、このアノテーションを付記したクラスのプロパティと、クライアントからリクエストで送られるJSONのkeyが対応している必要がある  
  
Spring Boot一般  
- `@SpringbootApplication`：Spring bootを活用したアプリのエントリーポイントとなるメソッド（通常はmainメソッド）に付記する。このアノテーションは、**そのクラスに`@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan`を自動で付け加える働きを持つ**  
  - `@Configuration`：そのクラスの中のメソッドで`@Bean`定義が行われることを示す  
    - `@Bean`：そのメソッドでインスタンス化したクラスが、シングルトンとしてDIコンテナに登録されることを示す。DIコンテナに登録したクラスは、別のクラス内で`@Autowired`で注入してアクセス可能。`@Component`との違いは、クラス定義とBean登録が切り離されているので任意の初期化手段でコンテナに登録できること  
    - `@Bean` vs `@Component`：https://stackoverflow.com/questions/10604298/spring-component-versus-bean  
    - シングルトンクラス：OOPの概念で、**一度に1つのインスタンスしか持てないクラス**のこと  
  - `@ComponentScan`：BeanとしてDIコンテナに登録すべきクラスがどのpackageにあるかを明示する。引数無しで付記した場合、Springはそのクラスのpackageとそれ以下の全てのpackageをスキャンしてBeanとして登録すべきクラスを探す  
  - `@EnableAutoConfiguration`：classpathに記述された依存関係に応じて、自動的にBeanを生成することをSpringに指示する  
  
Springを用いたWebAPI, Webアプリの処理の流れ：`SpringApplication.run`でDIコンテナが準備完了して各クラスが初期化され、アプリが起動されると、各エントリーポイントクラスが待機状態になる。そしてクライアントからHTTPリクエストが来ると、各リクエストが適切なクラスにDispatchされて処理が開始する
