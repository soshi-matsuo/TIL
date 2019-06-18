# Programming
## Python  
STTのAPI操作例  
```python
from ibm_watson import SpeechToTextV1
from os.path import join, dirname
# init STT endpoint
speech_to_text = SpeechToTextV1(   
 iam_apikey='{apikey}',
 url='{url}'
)
# add corpus to language model
with open(join(dirname(__file__), './.', 'corpus1.txt'), 'rb') as corpus_file:
speech_to_text.add_corpus(
'{customization_id}',
'corpus1',
corpus_file
)
# train language model
speech_to_text.train_language_model('{customization_id}')
```
  
osモジュール  
- `os.path.dirname`は、引数に与えられたファイルパスを機械的にDirname, Filenameに分割してDirnameの部分を返すだけ  
- 実行中ファイルのディレクトリ名を正しく取得するためには以下のようにする  
  - OK: `print(os.path.dirname(os.path.abspath(__file__)))` NG: `print(os.path.dirname(__file__))`  
- `os.listdir(<path>)`で、そのディレクトリ直下のファイル、ディレクトリの名前をリストとして返す


## Java
Map vs HashMap  
- Map：key:valueペアを格納するInterface  
- HashMap：MapをImplementしたClass  
- 一般的に、JavaでClassをインスタンス化して変数に代入する際は、代入先の変数の型を、初期化したいClassがImplementしているInterfaceの型として宣言する 
  - 継承している抽象クラスでもOK?
  - これにより、宣言したInterfaceをImplementしてさえいれば、実際に代入するObjectの型はなんでも良くなる  
  - [サンプルコード](https://stackoverflow.com/questions/1348199/what-is-the-difference-between-the-hashmap-and-map-objects-in-java)のように、HashMapを代入する変数をHashMap型として宣言してしまうと、実際に代入するObjectをHashMap以外に変更したくなった際に修正箇所が多くなる。変数の型宣言でMapを指定しておけば、実際に変数に代入されるObjectがTreeMapなどにに変わっても対応できる
  
ローカルクラス、無名クラス、ラムダ式  
- ローカルクラス：メソッドの処理中にクラスを宣言・利用できる仕組み。宣言するクラスは他のインターフェースを実装したものでもOK  
```java
public static void main(String[] args) {
  
  class Local implements Runnable {
    public void run() {
      System.out.println("Hello Lambda!");
    }
  }
  
  Runnable runner = new Local();
  runner.run(); // Hello Lambda!}
```

- 無名クラス：インターフェースを実装したローカルクラスの宣言を、省略できる仕組み（以下の例では単に`Runnable`クラスを初期化しているように見えるが、実際は`Runnable`インターフェースをImplementしたクラスを初期化している）  
```java
public static void main(String[] args) {

  Runnable runner = new Runnable() {
    public void run() {
      System.out.println("Hello Lambda!");
    }
  };
  
  runner.run(); //Hello Lambda!}
```

- 以上の無名クラスから、さらに `new Runnable(){}`, `public void run`を省略するとラムダ式になる。（だいたいRunnable, Callableを代入するために使われる）
  - 以下のように使われる。()が`call()`の引数を表し、->以降が`call()`の処理を表す  
    `Callable<T> task = () -> <some call>`
  - 参考：https://qiita.com/sano1202/items/64593e8e981e8d6439d3  
  
非同期処理  
- `ExecutorService`：スレッドプールを用いて非同期処理を行うためのオブジェクト。プログラム実行中に独立して他のタスクを扱わせることができる
- `ExecutorService`にタスクを非同期実行させるさせるには、`Runnable`または`Callable`インターフェースの実装を渡す。そのタスクの結果を扱いたい場合は`Callable`を渡す必要がある。
  - これは、`Runnabale`はネットワークサーバーの起動やディレクトリのモニタリングなどのために設計され、`Callable`は一回限りのタスクのために設計されたという設計意図の違いによるもの  
- `Future`：`ExecutorService.submit()`などが返すオブジェクトで、非同期処理の結果を扱うためのもの。一旦タスクがsubmitされると自動的にこのオブジェクトが返され、これを通じてタイムアウト上限を設定しつつ結果をgetしたり、実行途中のタスクをCancellしたりできる  
- ExecutorService, Callable, Futureイントロ：http://tutorials.jenkov.com/java-util-concurrent/executorservice.html
