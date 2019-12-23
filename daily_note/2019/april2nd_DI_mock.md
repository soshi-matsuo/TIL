# Learn
## Java(Spring)
- DIに関するアノテーション
  - `@Bean`：DIによって注入される中身を示すアノテーション。SpringのDIコンテナ(Spring Context)によって管理される。以下3つの定義方法がある
    - JavaConfig：`@Configuration`を付けたクラスでインスタンスを返すメソッドを定義し、そのメソッドに`@Bean`アノテーションを付けると、そのメソッドの返り値はBeanとして登録される
    - applicationContext.xml：設定ファイルとしてXMLファイルを使う
    - ComponentScan：`@ComponentScan(<path>)`のアノテーションを使い、`<path>`以下の`@Component`の付いたクラスをBeanとして登録する
  - `@Autowired`：DIによって注入される入れ物を示すアノテーション
    - Beanを注入したい場所でこれを付与することで、そのクラスと型の合うBeanが自動的に注入される
    - セッター、コンストラクタ、フィールドの3箇所で使うことができる
    - 注意点多数
      - **staticなフィールドに使うことは出来ない。** コンパイル時にクラス内のstaticメンバが初期化されるタイミングではDIコンテナの設定が完了していないこと＆クラスのstaticメンバは最初の一度しかロードされないことから、staticフィールドに`@Autowired`で注入しようとしても`NullPointException`が生じる
      - また、 **フィールドを`@Autowired`でDIしているクラスのインスタンスを`new`すると、そのインスタンスのフィールドはDIされず`Null`になる。** Spring Contextによる注入は`new`による初期化をカバーしていないらしい
      - JUnitによるテストを実行するときは、専用のアノテーションを付けないとInjectionが行われないようになっている
        - `@RunWith(SpringRunner.class)`と`@ContextConfiguration(classes=AppConfig.class)`：テスト実行クラスにこれを付与することで、テストの際もDIを実行した上でテストする。`SpringRunner`はSpring Contextをロードし、Beanをテストで扱うのを助ける
  
- テストについて
  - TestExecutionListener：SpringのTestContextManagerによるテスト実行イベントをリッスンし、テストの実行をサポートする
    - リッスン：通信機能を持つソフトが、外部からのアクセスに備えて待機すること
  - Mockito：モックオブジェクトを扱うためのライブラリ。
    - モック：ハリボテのオブジェクト。テスト対象クラスが別のクラスに依存していてテストしにくいときに、依存クラスの振る舞いを真似ることでテスト実施を助ける
    - `mock()`または`@Mock`でそのオブジェクトを完全にモックするか、`spy()`または`@Spy`で一部メソッドだけをモックする
    - `when(mockObj.someMethod(someParam)).thenReturn(someValue)`と書くことでモックの振る舞い＝オブジェクトのメソッドが呼ばれた時の返り値を固定でき、テストが簡単になる
    - `private`メソッドはMockitoのみではモック出来ないが、PowerMockを併用すればモック可能になる
    - `@MockBeans`：`@Autowired`とモックを併用するためのSpring側の機能。 **フィールドにこのアノテーションを付けると、テスト実行時にSpring Contextに登録されている同じ型のBeanを勝手にモックで置換してくれる。** 
  - PowerMock：Mockitoに機能を加えたライブラリ
    - Mockitoでは不可能な、`private`メソッドのモックが可能
    - 注意点
      - https://www.gwtcenter.com/difference-between-doreturn-and-when
      - Mockitoと併用する際の同名`static`メソッドの呼び出し方
  
- Stream：**Javaの入出力処理を司る、データをバイト単位またはテキスト単位で読み書きする機構。** 入力を担うInputStreamクラスと出力を担うOutputStreamクラスが根本の抽象クラスとして存在し、実際の処理は目的に応じた具象クラスが行なう
  - 標準入出力、ファイルへの入出力、WebAPIへの入出力など、あらゆる入出力（データのやり取り？）を抽象化するためのもの。Javaプログラムへのすべての入力はInputStreamを通じて行われ、逆も同じ。
  

## Other
- DI(Dependency Injection)
  - オブジェクトが成立するために **「必要な情報（Dependency）」** を、そのオブジェクト外で設定しておき、それを **「利用する際に送り込む（Injection）」** ことだと覚える。設定と利用の分離。
  - これによってオブジェクトを、再利用しやすいコンポーネント（部品）と捉えて実装できる  
  
- DIコンテナ：DI実現を楽にするためのフレームワーク  
  - 具体的には **「呼び出されるクラスのインスタンスを別ファイルで予め一元的に生成しておき、呼び出し側にはそのインスタンスを単に送り込むだけ」** という構図を実現する仕組み
  - 利点として、インスタンスが初期化される場所が1つ（DIコンテナファイル）に定まるので、引数の数や内容が変更されたとしても、DIコンテナでの初期化部分だけ修正すれば良くなる

## Ref
- Beanについて：https://dev.classmethod.jp/server-side/java/springboot-what-is-bean/  
- DI：https://www.atmarkit.co.jp/ait/articles/0504/29/news022.html  
- DIとDIコンテナ：https://qiita.com/ritukiii/items/de30b2d944109521298f  
- Spring Context：https://dzone.com/articles/what-is-a-spring-context  
- `@Autowired`とstaticメンバ：https://stackoverflow.com/questions/10938529/why-cant-we-autowire-static-fields-in-spring?noredirect=1&lq=1  
  
- Spring Contextと`new`：https://www.moreofless.co.uk/spring-mvc-java-autowired-component-null-repository-service/  
- JunitとMockitoの`@RunWith`アノテーションの使い分け：https://stackoverflow.com/questions/49635396/runwithspringrunner-class-vs-runwithmockitojunitrunner-class  
- JavaConfigでDIしているクラスをテストする方法：https://qiita.com/shin_hayata/items/6c8a375787bc7114833d  
  
- TestExecutionListener概要：https://www.logicbig.com/tutorials/spring-framework/spring-core/test-execution-listener.html  
- Listenerの使い分け：https://stackoverflow.com/questions/26125024/could-not-instantiate-testexecutionlistener  
- Springのレイヤーアーキテクチャ：https://www.yo1000.com/spring%E5%88%9D%E5%AD%A6%E8%80%85%E3%81%8C%E6%9C%80%E5%88%9D%E3%81%AB%E7%9F%A5%E3%82%8B%E3%81%B9%E3%81%8D%E8%B2%AC%E5%8B%99%E3%81%A8%E3%83%AC%E3%82%A4%E3%83%A4%E3%83%BC  

- Mockitoの基本アノテーション：https://www.baeldung.com/mockito-annotations  
- 簡単なMockitoの使い方：http://niwaka.hateblo.jp/entry/2015/09/19/171610, http://javatechnology.net/java/mock-injectmocks/  
- `@MockBean`：https://www.baeldung.com/java-spring-mockito-mock-mockbean
- 通常の`@InjectMock`と`@Autowired`併用：https://medium.com/@vatsalsinghal/autowired-and-injectmocks-in-tandem-a424517fdd29

- Stream：https://stackoverflow.com/questions/1830698/what-is-inputstream-output-stream-why-and-when-do-we-use-them
- MockitoでResponseを返そうとしたら`IllegalStateException`になった問題：https://stackoverflow.com/questions/19557672/unable-to-mock-glassfish-jersey-client-response-object
