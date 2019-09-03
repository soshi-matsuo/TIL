# Java
## Head First Java
### act12（インナークラス）
インナークラス  
- class内にclassを定義することで、outerClassの **`private`含む全てのメンバを利用できる**innerClassを定義できる
  - SubClassと違って、元のクラスの`private`メンバにアクセスするためにいちいちゲッターやセッターを使う必要がない
- １つのOuterClass内に複数のInnerClassを定義することで、1つのクラスの中で、Interfaceを違う内容で複数回Implementする事ができる
  - InterfaceのimplementをInnerClassに任せる
  - 本書のように、複数の同じイベント処理用のオブジェクトに各々別々の処理をさせたいときに有効
  
- InnerClassをインスタンス化するためには、すでにHeap上にOuterClassがインスタンス化されている必要があり、Heap上のOuterClassのオブジェクトを使ってInnerClassのオブジェクトを生成できる
  - この関係にあるInner, Outerのオブジェクトは相互のメンバにアクセスできる
- 基本的に、OuterClassの内部でのみInnerClassは作成される
  - `OuterObject.new InnerClass()`のようにして、OuterClassの外からInnerClassコンストラクタを呼び出すことも一応可能
- InnerClassは継承とは違う概念なので、OuterClassとInnerClassが全く別の継承ツリーに有ってもOK  

**HandsOn:** MIDIのNotesに合わせて適当なグラフィックを描画するプログラムを作る
- MIDIのサウンドなど、GUIコンポーネント以外もEventSourceとして扱える
- MIDIのメッセージとイベントを作成してControllerEventにリスナーを登録し、ControllerEventの発生に応じてグラフィック描画を行う  
  
- SwingによるGUIの大まかな流れ
  - JFrameのcontent, size, visible等を初期化
  - JButton、JLabelなどのGUI Componentを定義
    - JPanelを継承したクラスを作り、`paintComponent()`を好きなようにOverrideすることで任意の描画も可能
      - 任意のタイミングで`repaint()`を実行して再描画できる
  - GUIコンポーネントにEventListenerとなるクラスを登録することでイベント処理が可能
    - 全てを併用すると、MIDIなどのnon-GUIイベントをトリガーにしてGUIの描画を行ってアニメーションを作ることも可能（MiniMusicPlayerの例）
