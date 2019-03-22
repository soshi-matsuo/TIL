# Done  
- JavaのProgate終了  
- Javaで書かれたApache Solr用のAPIの機能改善
- STTのユーザー発話の取得
# Learn
## Python  
- reduce関数
  - `reduce(function, iterable, initializer(opt))`と引数を指定して呼び出す
  - `iterable`の先頭2つを取り出して`function`での処理を行い、その結果と`iterable`の次の要素を使って`function`の処理を行う、という流れを繰り返す
- max, sort関数におけるkeyパラメータ
  - 要素の比較を行う前に、リストなどの各要素に適用される関数を`list.sort(key=function)`のように指定することができ、その結果に基づいて大小比較が行われる

## Java 
## 基本言語仕様
- 環境設定
  - `$ /usr/libexec/java_home`コマンドを使ってどのバージョンにPATHを通すか設定できる
    - オプション無しで実行すると、インストールしてある中で最新のJDKのホームディレクトリのPathを表示する
    - `-V`オプションを付けると、インストール済みのJavaのディレクトリを一覧表示してくれる
    - `-v <version>`オプションで、該当するバージョンのJavaのディレクトリだけを表示してくれる
    - 実態は、/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands/java_homeへのシンボリックリンク
    - これを利用して、.zshrcの中でexport JAVA_HOME=`/usr/libexec/java_home -v "version"`と書いておくことで、任意のバージョンのJavaについてPathを通せる
    
- クラスとインスタンスについて
  - メンバ：オブジェクトに属する設定される変数、メソッドのこと
  - Javaのコンストラクタはクラス名と同名のメソッドに設定する
  - インスタンスを生成するときは`クラス型 変数名 = new クラス名(param1, param2...);`と、データ型の代わりにクラス型を宣言して、コンストラクタを使って変数へ代入する
  - アクセス修飾子：フィールドやインスタンスに付与するアクセス制限を表す
    - クラス外からもアクセス可能なpublic
    - 同じクラス内でのみアクセス可能なprivate
    - 同じクラス内＆サブクラスからのみアクセスを許すprotected
  - final修飾子：クラス、メソッド、フィールドについて、それぞれ継承、オーバーライド、値の変更ができなくなる  
  - クラスフィールド・クラスメソッド：staticのついているメンバ
    - クラスにとって固有の値（いくつインスタンスを生成したか等）や、そのクラスに基づくインスタンスが必ず共通の値を持つデータについて設定する
    - クラスメソッドの中からインスタンスにthisでアクセスすることはできない（クラスメソッドは各インスタンスと無関係に呼び出されるものなので、その中でthis.fieldなどとしてもどのインスタンスを指すのかプログラムが認識できない）
  - インスタンスフィールド・インスタンスメソッド：staticのついていないメンバ
    - 各インスタンスに属するメンバで、変数はprivate, メソッドはpublicにするのが一般的
  
- 継承について  
  - スーパークラスとサブクラス：継承元と継承先の関係で、複数クラスに共通するような抽象的な部分はスーパークラスとして切り出して、サブクラスの中で個別にカスタマイズすることで保守性を上げる
  - オーバーライド：スーパークラスのメソッドについて、サブクラス内で独自に上書きすること（オーバーロード=引数の型や個数の違う同名関数定義とは別）
    - `super.method()`とすると、スーパークラスで定義している上書き前の処理を呼び出すことができる
  - 抽象メソッド：スーパークラスの中に設定するインスタンスメソッドで、これがサブクラスのなかでオーバーライドされていなければエラーが起こる。詳細な処理はサブクラスごとに異なるがどのサブクラスにもそのメソッドを持たせたい、という場合に使う。また、これを持つスーパークラスは抽象クラスとなり、それ自体のインスタンスを生成することはできなくなる
  - インターフェース：抽象クラスと似ており、それを継承するクラスに対してメソッドの実装を強制することができる。抽象クラスとの違いは主に以下の二点。   
  1.「抽象クラス内には通常のメソッドも実装できるが、インターフェースではできない」  
  2.「あるクラスを定義する際に、複数のクラスを継承することはできないが、複数のインターフェースを継承することはできる」  
  - 多態性：サブクラスは、継承元のスーパークラスのクラス型としての性質を併せ持っており、サブクラス型のインスタンスは、スーパークラス型を宣言している変数に代入することができるという性質
  
- 型について  
  - `length`は配列の長さ(要素数)を表すintフィールドで、`length()`はString型の値の長さをint型で返すメソッド。**両者は別物**
    - String型は、char(文字)で構成された不可変の配列の役割を果たすオブジェクトだと考えるべき
    - lengthはメソッドではなくフィールドであり、不可変配列に対してのみ機能する
    - Stringもcharの不可変配列だと捉えることができるが、あくまでデータ型はオブジェクト(≠配列)なので、lengthフィールドを直接参照することはできない
  - さらに、Javaには不可変の配列型とは別に、Collectionインターフェースの配下としてList型やArrayList型などの可変配列（要素の追加や削除が可能な配列）が存在する。可変配列の要素数を取得するのは`size()`メソッドを使う必要がある
    - JsonArrayは、IterableインターフェースとJsonElementクラスのみを継承する可変配列であり、Collectionを直接実装しているわけではないが、要素数を取得するにはCollection同様`size()`メソッドを使う必要がある
    - ちなみに、CollectionインターフェースもIterableインターフェースを継承しているので、`size()`はIterableに属するとイメージするべきか？？
  - length, length(), size()の使い分けについて
    - https://programming-study.com/technology/list-length/
    - https://stackoverflow.com/questions/1965500/length-and-length-in-java

### ライブラリ・フレームワーク
- Spring frameworkのアノテーションについて
  - @Autowired：SpringにおけるDependency Injectionの実装手法で、インスタンスフィールドを定義する直前にこのアノテーションを付与すると、@Component（実際は@Service, @Controller, @Repositoryのいずれかが主）アノテーションのついたクラスの中から、定義したいフィールドと型が同じものをnewして代入してくれる
    - これによって、サブクラスの中などで明示的にnew Class()としなくてよくなる（依存性が疎になる？）
  - @Override：このアノテーションを付与されたメソッドは、スーパークラスやインターフェースのメソッドをオーバーライドして実装されていることを表す

- JAX-RS：JavaでRESTfulなWeb APIを実装するための仕様。Jerseyはこれの参照実装のこと
  - コード上では`import javax.ws.rs.<Class or method>;`としてパッケージをインポートする。以下が既存のAPIにリクエストを送るフロー
  1. `Client`のインスタンスが生成され、リソースURIとそれに対するクエリのパラメータを指定し、`client.target(URI).queryParam(~~)`の返り値として`WebTarget`クラスのインスタンスを生成する
  2. `webTarget.request(mediaType)`で、実際に送信するリクエストを構築して`InvBuilder`クラスのインスタンスとして返す
  3. `invBuilder.get()`で、実際にリクエストが送信される。返り値がAPIのレスポンスなので、`response`として格納
  
- Gson：JavaオブジェクトとJSONデータを相互に変換するためのパッケージ  
  - `JsonElement`, `JsonObject`がJsonデータを表す主なクラス  
  - JsonElement：それがJsonの要素であることを表すクラス。JsonObject, JsonArray, JsonPrimitive, JsonNullの4つはこれを継承している
  - JsonObject：それがJson形式のオブジェクトであることを表すクラス。JsonObjectはkeyが文字列でvalueが他のJsonElement(=Object, Array, Primitive, Null)になっている必要がある。  
  - この2つのクラスをaddメソッドなどで組み合わせていくことで、ネストした構造のJsonデータを定義することができる
  - Jsonデータから任意のデータを取り出す際はget系のメソッドを使う必要があるが、そのJsonデータがJsonElement型なのかJsonObject型なのか次第で、呼び出す際の引数の有無や、取り出せるデータの種類が変わってくることに注意（JsonObjectがいくつかのメソッドをオーバーライドしているため、同名で挙動が違うメソッドがいくつか存在する）  
  - JsonElementを文字列として取り出したい時について、`getAsString()`, `toString()`の2つのメソッドが考えられるが、`getAsString()`はプリミティブな値しか取得できない。詳細：https://stackoverflow.com/questions/34120882/gson-jsonelement-getasstring-vs-jsonelement-tostring

## Docker
- 基本コマンド
  - `$ docker build <dir> -t <name>`で、<dir>内のDockerfileに基づいて<name>と名付けられたdocker imageを生成する
  - `$ docker run <image>`で、docker imageに基づいた環境のコンテナを起動する
    - -pオプションで、Dockerfile内でEXPOSEしたポートと、ホストマシンの実際のポートをバインドする事ができる。つまり`$ docker run -p 8080:8080 some_image`としてコンテナを立ち上げて実際にホストマシンのブラウザでlocalhost:8080にアクセスすると、dockerコンテナのプロセスにアクセスできる
  - `$ docker ps`で起動中のコンテナ確認。-aオプションで停止中のものも含めて表示。-qオプションでコンテナIDのみを表示
  - `$ docker images`で生成したイメージを一覧表示
  - `$ docker stop <name or ID>`で起動中のコンテナを停止する。停止中のものを起動したければ`$docker start <name or ID>`でOK（何度も`run`しなくてよい）
  - `$ docker exec -it <name> <command>`でコンテナの中にアクセスすることができる。`<command>`の部分を`bash`や`sh`にすればホストマシンのターミナル操作と同じ感覚でコンテナの中を見れる
