# フロントエンド
## CSS
### HTMLでの要素修飾
- `<strong>`：タグ内文字列を強調  
- `<u>`：タグ内文字列に下線を引く  
- `<em>`：タグ内文字列を斜体に変更  
- `<s>`：タグ内文字列に打消し線を引く  
- `<hr>`：その位置に水平線を引く  
### 基本
- `box-shadow`：要素に対して影をつける  
- `opacity`：0~1の範囲で透明度指定  
- `ext-transform`：文字列の値そのものは変えずに、大文字、小文字などの表記を統一できる  
- `line-height`：テキスト1行の高さ（行間隔）を調整可能  
- 擬似要素：`<element>:<state>`の形でセレクタを書くと、elementがstateの状態になった時のCSSを指定することが出来る  
  - `:hover`：マウスオーバー時の状態を指定  
  - `::before`：指定したセレクタの直前位置を指定  
  - `::after`：指定したセレクタの直後位置を指定  
- `transform`：`:hover`などの擬似クラスと併用され、要素を動的に動かすのに使われる  
  - `scale()`：指定した倍率で要素を大きくする  
  - `skewX()`：X軸に対して指定した角度で要素を歪ませる（傾ける？）  
  - `skewY()`：Y軸に対して同上  
- `background: url()`：その要素の背面にurl()で指定した画像を配置する  
### 色指定
- `hsl()`：色指定を行う際に使える属性で、色相（色相環上の角度）、彩度（グレーの量）、明るさ（白または黒の量）を指定できる  
  - 彩度、明るさを調節することで色調（色の濃淡）を指定できる  
- `rgba()`：第4引数で透明度を指定可能なRGB指定  
- `background: linear-gradient()`：色のグラデーションを設定できる  
- `repeating-linear-gradient()`：指定したグラデーションのパターンを複数回リピートする  
### レイアウト
- CSSは、各HTML要素を長方形と見なして扱う  
- ノーマルフロー：デフォルトでは、以下の性質に応じて要素がレイアウトされる  
  - Block要素：自動的に改行して、新しい長方形の中にその要素が配置される（heading, paragraph, divなど）  
  - Inline要素：周囲の要素の長方形の中にその要素が配置される  
- position  
  - `position:relative`：ノーマルフローを基準に、`left`, `right`, `top`, `bottom`で設定した分だけ要素が動く  
  - `position:absolute`：position定義済みの最も近い親要素に対する相対的な位置を指定して、そこに要素を固定する  
    - デフォルト（他の要素が位置指定されていない場合）はbodyタグを親要素に取る  
    - ノーマルフローから要素が除外されるので、周辺の要素は`position:absolute`な要素を無視して配置される
  - `position:fixed`：ブラウザのウィンドウに対する相対的な位置を指定して、そこに要素を固定する  
    - absolute同様、fixedに指定された要素はノーマルフローから除外されるので、ノーマルフロー上の他要素からはこの要素は認識されなくなる  
    - absoluteとの大きな違いは、fixedされた要素はスクロールしても動かないという点  
- float  
  - `position:absolute|fixed`同様、コレが付与された要素はノーマルフローから除外される  
  - `float:left|right`：親要素の左または右にその要素を寄せる  
    - 要素の左右幅に大きく影響するので、width属性と併用されることが多い  
- z-index：前面〜背面の優先度指定を行う  
- block要素のmargin値をautoに設定すると、要素が中央に配置される  
  - `display:block`を指定したInline要素に対しても使える  
### アニメーション
- 基本的には、`animation-name`で名前、`animation-duration`で秒数を設定し、`@keyframes`でそのアニメーションが具体的にどのように推移するかを指定する、という流れで実現する  
- `:hover`などの疑似要素と併用することで、マウスオーバーでアニメーション起動、といった効果が実装できる  
  - `animation-fill-mode`をforwardsに設定することで、アニメーションの自動的なリセットを防げる  
- animation化された要素にpositionプロパティが設定されていれば、@keyframes内でtop, left, right, bottomの値を変化させることで、アニメーションの進行に従って要素を移動できる  
- @keyframes内でopacityを操作することでフェードイン・アウトが実装できる  
- `animation-iteration-count`を設定することで、指定した回数アニメーションを繰り返せる  
  - `animation-iteration-count: infinite;`でアニメーションを無限に繰り返せる  
- `animation-timing-function`で、指定されたdurationの中で、どれくらいの早さでアニメーションが推移するかを指定できる  
  - `animation-timing-function: cubic-bezier(x1, y1, x2, y2);`で、ベジェ曲線？を使ってより詳細に推移を指定できる  
## Ajax
- Ajax：ブラウザの動作を止めずにサーバーに非同期リクエストを送り、ページ全体を更新することなくレスポンスデータを表示する、という一連の流れを実装するための技術のこと  
- XHRまたはfetch APIでデータを取得し、そのデータを使ってDOMを更新するという流れ  
  - 目的のDOM要素のtextContentに、取得したJSONから抽出した文字列を挿入する  
  - htmlを構成する文字列を組み上げて、目的のDOM要素のinnerHTMLを更新する  
  - XHRでPOSTする場合は、onreadystatechangeイベントハンドラにCB関数を登録することで、リクエストの状態が変化した時に指定した関数を実行できる  
    - readyState：XHRオブジェクトの状態を表すプロパティ  
    - 参照：https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest/readyState  
- navigator API：各ブラウザに内蔵されているオブジェクトで、そのクライアントの緯度経度を保持しており、これを介してユーザーの位置情報を取得可能  
  - `navigator.geolocation.getCurrentPosition(callback)`で簡単にブラウザから位置情報が取れる  
