# Java
## Head First Java
### act12（インナークラス）
インナークラス  
- class内にclassを定義することで、outerClassのprivate含む全てのメンバを利用できるinnerClassを定義できる
- InnerClassをインスタンス化するためには、すでにHeap上にOuterClassがインスタンス化されている必要があり、Heap上のOuterClassのオブジェクトを使ってInnerClassのオブジェクトを生成できる
  - この関係にあるInner, Outerのオブジェクトは、相互のメンバにアクセスできる
- 基本的に、OuterClassの内部からのみInnerClassは作成される（`OuterObject.new InnerClass()`のようにして、OuterClassの外からInnerClassコンストラクタを呼び出すことも可能）
- １つのOuterClass内に複数のInnerClassを定義することで、同じInterfaceを違う内容でImplementする事ができる
  - 本書のように、複数の同じイベント処理用のオブジェクトに各々別々の処理をさせたいときに有効
