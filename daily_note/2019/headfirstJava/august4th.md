# Java
## Head first Java
### act9（GCとオブジェクトライフサイクル）  
Garbage Collection  
- Garbage Collection：実行中のプログラムのメモリ負荷が高まると、対象となっているオブジェクトを削除してメモリを確保する機構
  - **Heap上のオブジェクトは、（ローカル変数、インスタンス変数に関わらず）自らを参照する変数が存在しなくなるとGCの対象となる**
  - 「誰にも使われていないオブジェクトは無意味」という判断基準
  - オブジェクトと参照変数の結びつきが失われる具体的な事例
    - 参照変数のライフサイクル自体が終了する
    - 参照変数に別のオブジェクトが代入される
    - 参照変数にnullが代入される
  
  
変数のスコープとライフサイクル  
- ローカル変数
  - スコープ：変数が宣言された関数のStackフレームが、Stackの最上位に位置している間（関数実行中）、その関数内でのみ使用できる
  - ライフサイクル：変数が宣言された関数のStackフレームが、Stack内に残っている限り値は保持される（実行中か否かに関わらず値は生きている）
- Heap上のインスタンス変数は、それを保持するオブジェクトが存在する限り削除されない  

### act10（static, WrapperClass, 数字の扱い）  
`static`と`final`  
- Mathクラスのメソッドは全て`static`に指定されており、インスタンス変数を使う必要が無いようになっている  
  - コンストラクタが`private`になっているのでMathのインスタンスを作成することも出来ない  
  
- **`static`修飾子：クラスのメンバを、個別のインスタンスではなくクラスに紐付ける。**
  - メソッドを`static`に定義すると、そのメソッドが定義されているクラスのインスタンスが存在しなくても、`ClassName.staticMethod()`のようにして該当メソッドを直接呼び出すことができるようになる
    - インスタンスが格納された参照変数からstaticメソッドを呼び出すことも可能だが、staticメソッドはインスタンス固有の情報は扱わないと決まっているので、可読性が下がるだけ。Classから直接呼び出すべき
    - staticメソッドの中からインスタンス変数・インスタンスメソッドを直接使うことは出来ない（どのインスタンスのメンバを扱おうとしているのかコンパイラにはわからない）
  - 変数を`static`に定義するとクラス変数が定義でき、全てのインスタンスでその値が共有されるようになる   
    - static変数は、そのクラスがJVMによって最初にロードされるときに初期化される
    - JVMがクラスをロードするのは、プログラム内でそのクラスの（新しいインスタンスが作成される｜staticメンバが使用される）直前
  - non-staticなメソッドが1つでも存在していれば、そのクラスはインスタンス化する手段を備えている必要がある

- **`final`修飾子：そのメンバの値を変更不可能にする**
  - staticとfinalを併用することで、一度クラスがロードされると値が変更不可能になるような定数を定義できる
  - インスタンス変数やメソッド、メソッドの引数、クラスなどにも使える。メソッドに使うとOverRideの禁止、クラスに使うとExtendsの禁止を意味する　　
  
PrimitiveとWrapper  
- オートボクシング：Primitive→Wrapperへの自動型変換
- アンボクシング：Wrapper→Primitiveへの自動型変換
- Wrapperクラスの変数を宣言する際に初期値を入れない場合、nullとして初期化される
  - nullが入ったWrapperクラスをアンボクシングしようとするとNullPointExceptionが発生する
    - これは、アンボクシングの際には裏側で`wrapper.intValue()`などの変換用の関数が呼ばれているから
- Wrapperクラスは、String型を、それぞれが担当するPrimitiveの値に型変換するstaticメソッドを持っている
  - 各Wrapperの持つ`toString()`メソッドで、逆も可能  
  
書式の変換（フォーマット）  
- 数字のformatは`String.format()`で行う
  - 第一引数（フォーマット指定子）でフォーマットの書式を指定し、第二引数にフォーマット対象の数字を入れる
  - 第一引数はString型を取るので、通常の文字列内にフォーマット指定子を埋め込むような書き方も可能
    - その場合、%はフォーマット後の第二引数のプレースホルダーになる
  - フォーマット指定子は%から始まってd, f, x, cなどのtypeで終わる
    - 基本的には`%d`（10進数）, `%f`（小数）が多く使われ、後者は`%.1f`のように小数点以下を何桁にするかという精度インジケータと併用される
    - 日付書式のフォーマットには、tc, tA, tB, tdなどの2文字の指定子が使われる
    - フォーマット指定子が複数ある場合、2個目以降は`%<tc`のように＜を使うことで、直前のフォーマット指定子と同じ値をフォーマットする事ができる
- Javaで日付を扱う際、DateモジュールではなくCalenderモジュールを使うべき  

### act11（例外処理とその考え方）
例外処理の考え方
- あるコードをプログラム内で使用するとき、それが常に正しく動くとは限らず、実行時に想定外の状況に陥る可能性があるので、それをケアしておく必要がある
  - 例外をケアするためには、使おうとしているコードが起こしうる例外(検査例外)を事前に把握しておく必要がある
  - 各メソッドのシグネチャに`throws SomeCheckedException`の様に、そのメソッドが起こしうる検査例外を書く
  - メソッドが起こす例外は呼び出し元に返されるので、リスキーなメソッドを使おうとしている部分をtry-catchでラップする  
    - try-catchのどちらが実行されたに関わらず、finallyブロックの処理は**必ず**実行されるので、clean up用に便利
- 例外処理が正しく書かれていないとコンパイルは通らない（RuntimeExceptionは除く）  
- どのコードが例外をthrowしうるか、どのコードがそれをcatchするかを把握することが重要
- Exceptionも通常のObject同様継承関係が存在するので、throw-catchでポリモーフィズム的に幅広く例外処理できる
  - if-else ifと同様、特殊なものから先に該当するように書く必要がある
  
例外の無視  
- 例外の起こりうるメソッドをcallする際、try-catchで処理するだけではなく、更に上位の呼び出し元に処理を任せることもできる（例外発生メソッドを呼び出すメソッドのシグネチャにthrowsを宣言する）
- 最終的にはプログラムのどこかで例外をcatchしなければならず、Stack上の全メソッドが例外を無視した場合JVMはシャットダウンする  
  
検査例外と実行時例外  
- 狭義の例外（検査例外）はサーバーの死活やファイルの有無など、開発者側で制御できない状況をケアするためのものであり、対応する例外処理が必要
- 一方、RuntimeExceptionはコードロジックの欠陥に基づくエラーであり、正しく実装されていれば起こり得ないものであるため、これをケアするための例外処理コードは必須ではない

### act12（GUI、イベント処理）
GUIコンポーネントとイベント処理  
- Swingライブラリを使えば基本的なGUIコンポーネントを扱える  

- イベント処理は、GUIに起こるイベントをプログラム側で待機（Listening）し、イベント発生時にそれがプログラムに伝わって処理をトリガーする、という流れで行われる
  - GUIコンポーネント：プログラム＝イベントソース：リスナーの関係
    - イベントソース：キータッチ、クリックなどのユーザーの行動を扱えるオブジェクト。Eventオブジェクトを生成してリスナーに通知する
    - リスナー：イベントソースからイベントを受け取るためのInterface。イベントとリスナーは1対1
      - リスナーは`eventSource.addActionListener(this)`を呼び出して自身をリスナーオブジェクトとして登録し、イベントソースはイベントが発生すると`listener.actionPerformed()`を呼び出してリスナーにイベント発生を通知して処理をトリガーする  

- GUI上に描画を行う方法は主に3つ
  - `frame.getContentPane().add()`で、Frame上にウィジェットを追加する
  - graphicsオブジェクトでウィジェット上に2次元で描画する
  - `graphics.drawImage()`で画像を貼り付け
- JPanelを継承したClassを定義し、`paintComponent()`をオーバーライドすることで、ウィジェット自身の基本デザインを自由に定義できる
  - `paintComponent()`はシステムがウィジェットをロードする際に自動的にcallされるもの