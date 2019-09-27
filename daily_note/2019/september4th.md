# Java
## Head First
### act16（コレクションとジェネリクス、データ構造）
Collectionについて
- Collection：Collection InterfaceのSubInterfaceを実装した、複数のオブジェクトを要素としてまとめて保持するためのオブジェクトのこと
  - CollectionのSubInterfaceには、List, Set, Mapの3種があり、実際に使用するCollectionクラスはこれらのいずれかを実装している
- 主要なCollection
  - TreeSet：要素を重複排除＆昇順で保持する
  - HashMap：連想配列や辞書型
  - LinkedList：Stack, Queueの実装に有効
  - HashSet：集合型
  - LinkedHashMap：要素が追加された順序や、最後にアクセスされた要素のインデックスを保持する辞書（OrderDict？）
- ArrayListは最も頻繁に使われるCollectionの1つではあるが`sort()`メソッドを持たないので、順番を重視する場合はその他のCollectionのSubClassで代用するのも有効
  - ただし、必ずしも常にソートしておかなくてもいい場合には、TreeSetなどの自動ソート機能はオーバーヘッドになる（要素を追加するたびに何番目のインデックスに入るべきかを計算している）
  - オンデマンドでソートしたい場合は、`java.util.Collections.sort()`を使うのが有効
- `java.util.Collections`： **Collectionクラス群に作用するstaticメソッドをまとめたクラス** 
  - `sort()`に対してArrayList<String>を渡す際は正常に動作し、ArrayList<MyClass>を渡すとコンパイルエラーが生じる
  - これは、MyClassのどのインスタンス変数をソートの基準にすべきかわからないから
  - 後述のジェネリクスを理解することで、自作クラスを要素に持つCollectionを`sort()`に渡せるようになる  
  
ジェネリクスについて
- Collectionの型安全性を保つために使われる言語仕様
- ジェネリクス登場以前は、Collectionの要素の型を事前に定めておくことが出来ずあらゆるCollectionが`ArrayList<Object>`のように扱われており、要素を抽出する際もObject型として取得されていた
- ジェネリクスを使って生成したCollectionのオブジェクトを変数に代入したり、引数に取りたいときは、変数や引数の型を1段階抽象化しておく
  - ex. `List<Dog> dogs = new ArrayList<Dog>();`と、ArrayListを初期化して変数に代入する際はList型の変数を宣言して代入する
- ジェネリクスを利用するCollectionクラスとして、ArrayListのAPIドキュメントが好例
  - `<E>`はそのCollectionが保持して返す要素の型を抽象的に表している
  - Doc中の`<E>`部分が、初期化する際に与える実際の型に置換されると考えてOK。型引数を初期化時に与えることでそのCollectionの`add(<E> o)`が`add(<MyClass> o)`に更新されるので、**宣言した型を持つ要素以外を与えることができなくなる**
- メソッドの定義にもジェネリクスが使われることがある
  - クラス宣言時に使われた型引数を、そのままメソッドの引数の型引数として使える
  - そのメソッドが引数に受け取るCollectionが保持すべき要素の型を、シグネチャ内にジェネリクスを書くことで定義できる
    - ex. `public <T extends Animal> void sampleMethod(ArrayList<T> list)`とすると、`sampleMethod`の引数`list`は`Animal`を継承or実装したクラスのオブジェクトを要素に持っていなければならない
    - `public <T extends Animal> void sampleMethod1(ArrayList<T> list)`の引数`list`は、`Animal`またはそのSubClassを要素に持つArrayListを取るが、`public void sampleMethod2(ArrayList<Animal> list2)`の引数`list2`は、`Animal`そのものを要素に持つArrayListしか受け取ることが出来ない(詳細後述)
- ジェネリクスにおける`extends`はSubClassの継承とInterfaceの実装いずれの意味も併せ持つ
- 上記のように型引数が定義されたクラスやメソッドをジェネリックなクラス/メソッドと呼ぶ  
  
`Collections.sort()`について  
- `Collections.sort()`のAPIDoc上のシグネチャは`public static <T extends Comparable<? super T>> void sort(List<T> list)`となっており、引数`list`の要素は`Comparable`を実装しているべきだと分かる
  - `Comparable`：オブジェクトどうしの比較を可能にするクラスで、これを実装していないオブジェクトはソートの対象に取れない
  - `Comparable`の実装の際は、クラスの内部で`compareTo()`をオーバーライドして、そのクラスのオブジェクトどうしをどのように比較するか定義する
    - ex. 本書中の例だと、String型のインスタンス変数titleを使ってアルファベット順に並べたいので、Stringクラスが実装している`compareTo()`を流用すればOK
- あるクラスを任意の基準でソートしたい場合には、`sort(List<T> list, Comparator<? super T> c)`とオーバーロードされたバージョンを使えば良い
  - `Comparator<T>`は`compare(T o1, T o2)`メソッドを持つインターフェースなので、これを実装するソート用のクラスを複数定義すれば、毎回任意の基準でソートを実行できる
  - `Comparator`を渡してソートを実行する場合は、オブジェクトの持つ`compareTo()`ではなく`Comparator`の`compare()`に基づいて順序が決定される  
  
Collection API(Collection Framework)の概略  
- Collection(IF)
  - Set(IF)
    - SortedSet(IF)
      - TreeSet(Class)
    - LinkedHashSet(Class)
    - HashSet(Class)
  - List(IF)
    - ArrayList(Class)
    - LinkedList(Class)
    - Vector(Class)
- Map(IF)
  - SortedMap(IF)
    - TreeMap(Class)
  - HashMap(Class)
  - LinkedHashMap(Class)
  - HashTable(Class)
- Map InterfaceはCollectionを直接実装してはいないが、Collection APIの一部として扱われる  
  
Setについて
- オブジェクトの重複を許さないデータ構造
- 重複の判断基準は`Object.equals()`がT/Fのどちらを返すのかであり、このメソッドは内部で`Object.hashCode()`を呼び出して、その返り値が同じであればtrue, 違えばfalseを返す
- 自分でクラスを定義してそのオブジェクトを扱う際には、「そのオブジェクトのどの属性が一致していれば重複と言えるのか」に基づいて、`equals()`と`hashCode()`をオーバーライドしなければならない
- 「オブジェクトが同じ」には2つの意味がある
- 参照しているオブジェクトの一致
  - **異なる複数の参照変数が、Heap上の同一のオブジェクトを参照しているとき**
  - これらの変数から`hashCode()`を呼び出すと同じ値が返る（デフォルトではオブジェクトのメモリアドレスに基づいて値を返すので）
  - `==`演算子での比較はこれを検証している
- 異なるオブジェクト間の値の一致
  - **異なる複数の参照変数が、各々Heap上の別のオブジェクトを参照しているが、それらのオブジェクトが意味的に等しいと言えるとき**
    - ex. 複数のSongオブジェクトがHeap上に存在していても、title属性さえ一致していれば同じオブジェクトと見なせるとした場合
  - この状態を機能として実装するためには`equals()`と`hashCode()`をオーバーライドしなければならない  
- HashSetの重複検証は、まず`hashCode()`の返り値が同じかどうかを確かめ、次に`equal()`がtrueを返すかどうかを確かめるという2段階になっている
  
Mapについて
- Mapオブジェクトを標準出力にプリントすると、{key=value}という形式で出力される  
  
    
ジェネリクス詳細＆ワイルドカード  
- 引数にArrayList<Animal>を取るメソッドに対して、ArrayList<Dog>のようなコレクションを渡そうとすると、コンパイルエラーが生じる（AnimalのSubClassがDog）
  - これは **引数に渡されるコレクションの型安全性を守るため**
    - このようなメソッドでは引数に受け取ったコレクションに要素を追加する操作が生じうる。また、ArrayList<Animal>を引数に想定したメソッドなので、あらゆるAnimalのSubClassを追加することが考えられる。しかし、そのような操作が起こりうるメソッドでArrayList<Dog>を受け取ることを許してしまうと、「Dogしか要素に持たないコレクションに対してCat型の要素が混入してしまう」問題が生じる
  - SuperClass配列を受け取るよう定義されたメソッドに対して、SubClass配列を引数に渡した場合にコンパイルエラーが出ないのは、配列の要素の型はコンパイル＆ランタイムの2回検査されるから
    - コレクションの場合はランタイムでの型検査は行われないので、コンパイル時の要素の型安全性の要求が厳しい
- 後述の通り、変数を初期化する際も同じで、**宣言した変数のジェネリクスの型と、実際に代入するコレクションのジェネリクスの型は全く同じである必要がある** 
  
- `public void sample1(ArrayList<MyClass> list)`のシグネチャでは、上記の理由でMyClassのSubClassを要素に持つArrayListを引数に渡すことは出来ない
- これを解決するためのオプションが**ジェネリクスのワイルドカード**
  - `public void sample2(ArrayList<? extends MyClass> list)`と定義することで、MyClassを継承or実装するSubClassを要素に持つArrayListも引数に渡せるようになる
  - `public ArrayList<T extends MyClass> void sample3(ArrayList<T> list)`も同じ意味になる。複数のCollectionを引数に渡す際はこちらのほうが書く量が減る
  - この記法を使うと、コンパイラは、SubClassを要素に持つCollectionを受け取ることを許す代わりに、そのCollectionに要素を追加することを禁止する→実行時の型安全性が保たれる
- **ジェネリクスの型引数は、ワイルドカードを使わない限り、宣言した通りの型しか受け入れない**
  - `List<Animal> list = new ArrayList<Animal>();`（ArrayListをListとして宣言）は可能だが、`ArrayList<Animal> animals = new ArrayList<Dog>();`（ArrayList<Dog>をArrayList<Animal>として宣言）は不可能
