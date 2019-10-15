# Java
## Head First Java
### act12（インナークラス）
インナークラス  
- class内にclassを定義することで、outerClassの **`private`含む全てのメンバを利用できる**innerClassを定義できる
  - SubClassと違って、元のクラスの`private`メンバにアクセスするためにいちいちゲッターやセッターを使う必要がない
- １つのOuterClass内に複数のInnerClassを定義することで、1つのクラスの中で擬似的に、Interfaceを違う内容で複数回Implementする事ができる
  - InterfaceのimplementをInnerClassに任せる
  - 本書のように、複数の同じイベント処理用のオブジェクトに各々別々の処理をさせたいときに有効
  
- InnerClassをインスタンス化するためには、すでにHeap上にOuterClassがインスタンス化されている必要があり、Heap上のOuterClassのオブジェクトを使ってInnerClassのオブジェクトを生成できる
  - この関係にあるInner, Outerのオブジェクトは相互のメンバにアクセスできる
- 基本的に、OuterClassの内部でのみInnerClassは作成される
  - `OuterObject.new InnerClass()`のようにして、OuterClassの外からInnerClassコンストラクタを呼び出すことも一応可能ではある
- InnerClassは継承とは違う概念なので、OuterClassとInnerClassが全く別の継承ツリーに有ってもOK  

**HandsOn:** MIDIのNotesに合わせて適当なグラフィックを描画するプログラムを作る
- MIDIのサウンドなど、GUIコンポーネント以外もEventSourceとして扱える
- MIDIのメッセージとイベントを作成してControllerEventにリスナーを登録し、ControllerEventの発生に応じてグラフィック描画を行う  
  
- SwingによるGUIの大まかな流れ
  - JFrameのcontent, size, visible等を初期化
  - JButton、JLabelなどのGUI Componentを定義
    - JPanelを継承したクラスを作り、`paintComponent()`を好きなようにOverrideすることで任意の描画も可能
      - 任意のタイミングで`repaint()`を実行して再描画できる
  - GUIコンポーネントに`someComponent.addEventListener()`でEventListenerとなるクラスを登録することでイベント処理が可能
    - 全てを併用すると、MIDIなどのnon-GUIイベントをトリガーにしてGUIの描画を行ってアニメーションを作ることも可能（MiniMusicPlayerの例）
    
- LayoutManagerについて
  - Swingで定義されるGUIコンポーネントは全て`javax.swing.JComponent`を継承している
  - コンポーネント内には他のコンポーネントを配置できる
    - 基本的に、この用途にはFrameやPanelが使われる
    - 他のコンポーネントを含むコンポーネントは、LayoutManagerオブジェクトを用いて内包コンポーネントの位置やサイズを制御する
    - 個々のコンポーネントのLayoutManagerの制御は直下のコンポーネントにのみ影響する
  - LayoutManagerごとに適用するルールは異なる
    - BorderLayout：FrameのデフォルトのLayoutManager。エリアを5分割する
    - FlowLayout：Panelのデフォルト。コンポーネントを、左から右へ余白を埋めるように配置する
    - BoxLayout：FlowLayoutと似ているが、1行につき1コンポーネントを上から下に垂直的に配置する
    - LayoutManagerに応じて、コンポーネントのサイズ設定をどの程度反映させるかも異なる
    - `setLayout()`でLayoutManagerを変更することも可能
