# Java
## Head First Java
### act14（シリアライゼーションとファイル入出力）
オブジェクトの保存  
- あるオブジェクトのインスタンス変数などを維持したまま、外部にファイルとして保存する方法は、シリアライズor「直接ファイルにオブジェクトの状態を書き込む」の主に2種類ある
  - 保存したオブジェクトがJVMによって使用されるかどうかが判断基準  
  
シリアライズ  
- ConnectionStreamとChainStreamを組み合わせて行われる
  - ConnectionStreamは、ファイルやネットワークに対してデータをやり取りするための通信経路を表すクラス
  - ChainStreamは、ConnectionStreamが扱えるような形式にJavaプログラム上のデータを変換する（Serialize）役割を担うクラスで、単体では使用できない
- オブジェクトがシリアライズされるときは全てのインスタンス変数がシリアライズされる
  - もしオブジェクトが、他のオブジェクトへの参照をインスタンス変数として持っていた場合、その参照されているオブジェクトも連鎖的にシリアライズされる
  - `static`変数はインスタンスではなくクラスに属するのでシリアライズされない
- 自作クラスをシリアライズ可能にしたい場合は`Serializable`インターフェースを実装する
- シリアライズ処理の結果は、DBのトランザクションのように、対象オブジェクトが正しくシリアライズされるか全くされないかのどちらかにしかならない
  - シリアライズしたいオブジェクトが`Serializable`でないオブジェクトをインスタンス変数として参照していた場合、シリアライズは失敗する
  - オブジェクトは、ネットワークの接続状態やスレッドの状態などをインスタンス変数として持つことがあるが、これらは特定の実行環境依存な情報であり、保存しても意味がないので保存できないようになっている
  - シリアライズ処理をスキップしたいインスタンス変数がある場合は`transient`修飾子をつける
    - `transient`に指定されたインスタンス変数は、デシリアライズされたときにnull, false, 0などが入る
    - オブジェクトの状態が`transient`な変数の値に依存している場合は、シリアライズ以外の手段で別個にその値を保存する必要が生じる  
  
デシリアライズ  
- シリアライズの逆のフローをたどる
- 読み込もうとしているオブジェクトのクラスが、読み込み側のプログラムの中に見つからない、ロードできない場合例外をthrowする
  - クラスごとシリアライズしないのは、オブジェクトの状態の保存や送信という目的に照らして、オーバーヘッドがデカすぎるから
  - シリアライズしたオブジェクトをネットワークで送信する際に、そのオブジェクトのクラスが存在するURLの情報を併せて送ることができ、受け取り側でクラスが見つからない場合はそのURLを使ってクラスをロードすることが可能
- 新しいオブジェクトのための領域がHeap上に確保されるが、読み込もうとしているオブジェクト自身のコンストラクタは使われない
  - シリアライズされたオブジェクトの継承ツリーを遡って、`Serializable`ではないSuperClassにたどり着くと、そこから上のクラスのコンストラクタが実行される
- オブジェクトがHeap上に生成されると、シリアライズされた情報を元にオブジェクトの状態が復元される  
  
ファイルへの書き込み  
- `java.io.File`は記録装置上のファイル名、ファイルパスを司るクラス（ファイルの内容は扱わない）
  - Fileオブジェクトを経由してWriterやStreamにファイルを渡すことで、直接Stringでファイル名を渡すよりも安全になる（パスの有効性などが確かめられる？）
- `BufferedWriter`はChainStreamの一種で、`FileWriter`などのConnectionStreamと併せて使う
  - ファイルへの書き込みを行う前にBufferを介することで、Bufferの容量がいっぱいになるor`BufferedWriter.flush()`が実行されるまで、実際の書き込みが行われなくなる
  - `BufferedWriter`を介さずにファイルへの書き込みを実行する場合、`FileWriter.write()`を実行するたびにディスクへのデータ書き込みが行われるが、この処理はボトルネックになる
    - 書き込みたい内容が確定するまでBufferにデータを持っておいて、ディスクへの書き込みは一回にまとめる方が効率が良くなる
    
ファイルからの読み込み  
- シリアライズ：デシリアライズ同様、書き込みと逆のフローをたどる
- whileループで対象ファイルから読み込むべきデータがなくなるまで`readLine()`を繰り返す、という手法が一般的  

オブジェクトの状態管理  
- シリアライズ→デシリアライズの過程で、そのオブジェクトのクラスに対して特定の種類の変更があった場合、デシリアライズは失敗する
  - ex. インスタンス変数の型変更や削除、`transient`化など
- JVMはシリアライズの際に、シリアライズ対象オブジェクトの属するクラスに対してversionIDを記録する
  - これは`serialVersionUID`と呼ばれ、クラスの構造を元に計算される
- デシリアライズの際は、オブジェクトがシリアライズされた時点のクラスIDと、デシリアライズ時点のクラスのIDを比較し、IDが一致して互換性があるとみなされるとオブジェクトの復元が行われる
  - 通常、クラスに変更を加えると自動的にIDが更新されるためデシリアライズは失敗するが、クラス内でIDを定数として明示的に宣言しておくことで、クラスの変更に伴うIDの自動更新を防ぐことが出来る
  - このようにIDを固定する場合、シリアライズされたオブジェクトとそれが属するクラスの互換性をデベロッパー自身で保つ必要があり、もしクラス構造を変更した場合、手動でクラスのIDを更新しなければならない
  
### act15（ネットワーキングとスレッド処理）
socket通信
- 通信とは→2つのソフトウェアがお互いにデータを送る手段を確立している状態
  - サーバーとsocket通信を行うには、IPアドレス（ネットワーク上のサーバーの位置情報）とTCPポート番号（サーバー上で動いているプログラムを特定する番号）を知っていれば良い
  - ポート番号は、1024~65535の間であればサーバープログラム側で任意に決定して良い
    - 0~1024はその他のよく使われるサービス（HTTP, FTP, SMTPなど）のために予約されている
- Javaプログラムでsocket通信を使ってメッセージ受信を行う際は、ファイル同様`BufferedReader`が使える
  - socketでの送信の際は、`BufferedWriter`を介さずに`PrintWriter`が直接使われる事が多い（リアルタイムチャットでは、書き込みは一度に1つのStringを送信できればOKだから）
  - 送受信ともに`Socket`クラスの持つ`InputStream`, `OutputStream`にチェインする必要がある  
  
**Hands On:** リアルタイムチャットサーバー＆クライアントの実装  
- サーバー側でポート番号を指定して`ServerSocket`オブジェクトを初期化し、`ServerSocket.accept()`を実行してクライアントからの接続を待機する
- クライアント側でサーバーコンピュータのIPと`ServerSocket`が実行されているポート番号を指定して`Socket`オブジェクトを生成する
- クライアントからの接続要求を受け取ると、`ServerSocket.accept()`はクライアントのIPとポートを読み込んだ`Socket`オブジェクトを サーバープログラム内に返す
- クライアント、サーバー両者の`Socket`オブジェクトを介してデータのやり取りが可能になる
- 他のクライアントがサーバーにメッセージを送ると、即座に他のクライアントにもそれが読み込まれるようなリアルタイムチャットアプリ
- Streamへの書き込みを行うと同時に、Streamからの読み込みを行う必要がある→ **マルチスレッド**  
  
マルチスレッド  
- スレッド：プログラムの実行主体と考えてOK。1つのスレッドは各々が固有のcall stack（関数呼び出しのStack）を持つ  
  - プログラムをマルチスレッド化することで、サーバーからデータを読み込みながら、同時にサーバーに対してデータを書き込むことができる  
- `Thread`オブジェクトを生成し、`start()`で新しいスレッドを起動することができるが、スレッドに実行させるジョブが指定されていないと何も起こらない  
- JVMは通常、`main()`を起点にしてスレッド（メインのコールスタック）を立ち上げてプログラムを実行する  
- マルチスレッドの場合でも、OS上で起動しているJavaプログラムのプロセスは1つのまま
- スレッドが起動すると、JVMはmainスレッドと新しいスレッドのStack最上位の処理を、交互に高速で処理していくことで、擬似的に同時に複数の処理を行っている（タイムスライス方式）
- `Runnable`インターフェースを実装したクラスを`Thread`コンストラクタにわたすことで、`Thread`オブジェクトに実行させるジョブを定義できる
  - `Thread.start()`によって、ただのオブジェクトが実行対象スレッドとしての機能を持つ、と考えてOK  
  - `Runnable`インターフェースは、新しいスレッドの実行は必ず`run()`から始まるという規約を敷いている
- `Thread.start()`でスレッドは実行可能状態に入るが、どのスレッドがいつ実際に実行されるかを決定するのはJVMのスレッドスケジューラ
  - スケジューラの動作を完全に制御できるAPIは存在しない。ホストマシンやJVMの違いによってもスレッドの実行順は変わりうる
  - スレッドが実行状態に入ると、ジョブとして渡した`Runnable`なオブジェクトの持つ`run()`メソッドが実行され、それを起点に新しいコールスタックが実際に動き出す
  - スレッドは基本的に実行状態と実行可能状態を行き来するが、他のスレッドの終了を待ったり、`sleep()`が呼ばれたり、Streamからのデータ読み込みを待ったりするときに、一時的に実行状態から待機状態に入ることがある  
  - スレッドスケジューラの采配がどのようになるかは実行してみなければわからないが、`Thread.sleep()`を呼び出すことで実行中のスレッドを一定時間強制的に待機状態にすることができ、その間は他のスレッドが実行状態に繰り上げられる
  - `sleep()`は検査例外をthrowすることになっているのでtry-catchが必要（通常の呼び出しなら、実際に例外が生じる可能性はほぼない）
  - `sleep()`終了後のスレッドは実行状態ではなく実行可能状態になり、再びスケジューラによる実行指示を待つことになる
- 一度`run()`を実行完了した`Thread`オブジェクトに対して再び`strat()`を呼び出すことは出来ない（実行完了してもHeap上にオブジェクトとしては存在するが、スレッドとしての機能は失われる）  

排他制御  
- スレッド処理の問題点：各スレッドは、素の状態では他のスレッドがどのような操作を行ったかを一切気にしない（排他処理をしない）ので、オブジェクトの状態などのデータが壊れてしまう
  - マルチスレッドを適切に動かすためには、あるスレッドがオブジェクトにアクセスして処理を行っている最中は、他のスレッドはそのオブジェクトにアクセスできない、という排他制御を実装する必要がある
  - ここでの排他制御とは、問題が起こりうる処理を **atomic** にすること
    - atomic：中断・分割不可能な一連の処理（トランザクション処理？）
- `synchronized`キーワードを使って、「複数のスレッドによって呼び出される可能性がある、オブジェクトを操作するメソッド」（マルチスレッドによってデータが壊れうるメソッド）に排他制御をかけることができる
  - メソッドを`synchronized`にすると、スレッドAがそのメソッドを実行中に`sleep()`などで待機状態に入ったとしても、スレッドBはスレッドAの処理が完了するまでそのメソッドを呼び出せない
  - 排他制御は`synchronized`なメソッド1つ1つに対してかかるのではなく、`synchronized`メソッドを持つオブジェクトに対してかかる。つまり、オブジェクトが複数の`synchronized`メソッドを持っていたとして、スレッドAがそのうち1つを呼び出している間は、スレッドBはそのオブジェクトのあらゆる`synchronized`メソッドにアクセスできない
  - `synchronized`をクラスメソッド（`static`メソッド）に対して宣言することも可能
- `synchronized`による排他制御は、以下の理由により無闇に使うべきではない
  - パフォーマンスの低下（`synchronized`自体のオーバーヘッド＆オブジェクトの並行処理を禁じることによるマルチスレッドの利点の減少）
  - デッドロックが発生しうる（スレッドA,Bが互いに、相手がアクセスしたがっているオブジェクトへのアクセス権を保持したまま膠着してしまうこと）
- メソッドよりも小さいブロック単位で`synchronized`にすることもできる