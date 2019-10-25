# Java
## Gradleを使ったビルドとアプリの起動
- gradlewでビルドを行う場合、バージョン情報などは`gradle-wrapper.properties`を参照する  
  - `build.gradle`の中のラッパータスクは、`gradle wrapper`でラッパーを初期化する際に実行されるタスクであり、ラッパーの情報を更新したい場合は直接プロパティファイルを編集する  
- `build.gradle`のあるディレクトリで`gradle build`を実行すればビルドが行われる  
  - ビルド後のjarファイルの在り処→pjroot/build/libsディレクトリに生成される
- `build.gradle`内でapplicationプラグインが使われていれば、javaコマンドを使わずに`gradle run`または`gradle bootRun`でアプリが起動できる
  - terminalの出力がexecuting ~%...で止まるのは無視してOK。Gradle側ではtearDown処理まで含めて１００％として数えている

## Spring bootの設定について概略
- 参考：https://www.baeldung.com/properties-with-spring
- Spring3.1以降`@PropertySouce`アノテーションが有効になり、このアノテーションの引数に渡した設定ファイルを、クラス内でEnvironment型のフィールドとして`@Autowired`で読み込む事ができる  
- `application.properties`：/src/main/resourcesに配置することで、デフォルトで設定ファイルとして扱われるファイル  
  - Production, Stagingなど、環境固有の設定項目が存在する場合は`application-environment.properties`を同じディレクトリに作成する  
    - 一般設定と環境固有設定はどちらも読み込まれ、同じ項目については環境固有設定が優先して適用される  
  - 配置すべき外部設定ファイルとしては`application.yml`（YAML形式）もサポートされており、使い方は`application.properties`とほぼ同じだが、`@PropertySource`と併用できない  
  - `application.yml` or `application.properties`はファイル内で同名キーが存在した場合は単に後勝ちする
- src/test/resourcesに設定ファイルを配置することでテスト時に優先的に適用される設定を定義できる  
- `@ConfigurationProperties`をクラスに付記すると、そのクラスの同名フィールドの値として設定ファイルの項目がマッピングされる（参考：https://www.baeldung.com/configuration-properties-in-spring-boot）  
  - 同一のprefixを持つ、階層構造になった設定と相性がいい  
  - 各フィールドへの設定値の注入は、クラス内のSetterを使って行われるため、Setterを定義するかlombokのアノテーションを使う必要がある  
  - 他と同様、src/main/resourcesに配置した設定ファイルが使われる
  - 用例  
    - POJO（Plain Old Java Object）として設定ファイルの内容を保持するための独立したクラスを用意して、`@Configuration`、`@ConfigurationProperties`を併用することで、設定値を保持するクラスをDIに登録する  
      - `@Configuration`アノテーションは、表記はよく似ていてるが、コレ自体は`@Bean`定義用のメソッドを内包できるように`@Component`アノテーション（クラスをBeanとしてDIに登録するためのアノテーション）を拡張したものでしかない  
    - `@EnableConfigurationProperties`と併用することで、よりシンプルにDI登録できる  
      - 実際に外部設定値を活用するクラスに`@EnableConfigurationProperties`を付けて引数に`@ConfigurationProperties`付きクラスを渡すと、`@ConfigurationProperties`クラスで読み込んだ外部設定値を`@Enable~`クラス側で簡単に呼び出せる（参考：https://www.baeldung.com/spring-enable-config-properties）  
- jarファイル実行時のコマンドライン引数として渡す
- 環境変数を設定してjarファイルを実行する
- `RandomValuePropertySource`による設定値のランダム化

## Bean Validation
- 参考：https://www.baeldung.com/javax-validation
- ユーザー入力の有効性の検証のデファクトで、（Nullでないか等）JavaBeansのフィールドが特定の要件を満たしているかを検証する
  - ポピュラーな参照実装としてHibernate Validatorがある
  - エラー文を書く際にExpression Languageが併用される事が多い
- フィールドにアノテーションを付けた後は、Validatorインスタンスの`validator()`にBeanを渡すことで検証を行い、エラーメッセージのSetを返り値として取得する
- **JavaBeansは必ずしもSpring Beansと同じではない**（参考：https://stackoverflow.com/questions/21866571/difference-between-javabean-and-spring-bean）
  - Spring以外の文脈でBeanという言葉が使われていた場合、単に以下の性質を持つクラスを指す
    - `public`な引数なしのコンストラクタを持つ（デフォルトコンストラクタの存在）
    - Getter, Setterを使ってプロパティへのアクセスを行う（カプセル化）
    - `java.io.Serializable`インターフェースを実装している（シリアル化可能）
