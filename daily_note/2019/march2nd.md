# Done
- 基本情報技術者試験の勉強
- CheckIO * 8  
- Paiza * 2
- Pythonのjsonモジュールを用いたWatson Assistantの会話モデルの修正
- Watson Explorer Analytical Componentsチュートリアル

# Learn
## FE
- 2進数、論理回路、デジタルデータ、CPU、メモリ、補助記憶装置について  
- I/O機器、OSについて。特に記憶管理   
- DBについて。特に正規化
  - 第一正規化→繰り返しを別レコードに分離
  - 第二正規化→複合キーとなっている部分に対して部分関数従属なカラムを分離
  - 第三正規化→主キー以外の部分に対して部分関数従属なカラムを分離
## Python
- 関数定義の際、引数の頭文字に\*, \*\*をつけると可変長引数（任意の要素数の引数）を受け取れるようになる
  - \*argsを指定した場合、複数の引数をタプルとして受け取る  
  - \*\*kwargsを指定した場合、複数のキーワード引数を辞書として受け取る  
  
- collections.Counterのmost_common()メソッドで、(要素, 出現回数)のリストが得られる  

- リストへの要素の追加について  
  - 内包表記で書けず、for文でlist.appendを使いたい場合は、appendをループの外で前もって参照し、変数に代入しておくのが良い  
  - list.append(obj)：listの最後に、objを要素として追加する  
  - list.extend(another_list)：listの最後に、another_listの要素をすべて追加する  
  - list.append(another_list)とすると、listにanother_listの要素が追加されるのではなく、another_listというリストオブジェクトそのものが1要素としてlistに追加される  
  
- リスト内包表記＆再帰＆三項演算子を使ったflatten関数の実装  
  - 三項演算子  
    - `(if文がTrueのときに評価される式) if 条件式 else (if文がFalseのときに評価される式)`という構造でif文を一行で書ける  
    - elifを表したい場合は、else以下の部分を更に三項演算子でネストするように書く（可読性☓）  
    - リスト内包表記で使う場合、後置if文と混同しないよう注意  
    - 参考：https://note.nkmk.me/python-if-conditional-expressions/  
  - リスト内包表記  
    - 三項演算子と併用することで、イテレータの要素に対して条件分岐で処理した結果を使って、新しいリストを生成できる  
  - 再帰関数  
    - 自分自身を再帰的に呼び出し続ける関数　　
    - 再帰終了条件を持ち、状態を変えながら終了条件へ進む  
  
- JSONファイル周りの基本的な使い方の復習  
  - これが出来ればJSONファイルの簡単な一括修正などは可能  
  - json.loadでjsonファイルを辞書オブジェクトとして読み込み  
  - json.dumpで辞書をjsonファイルとしてファイルオブジェクトに書き込み　　
  
- reモジュール  
  - `re.compile('pattern')`とすることで、patternについての正規表現オブジェクトを生成可能。`match()`,`sub()`などのメソッドを、マッチさせたいパターンについての引数を省略して使える
  
## Java  
- Progateでの基本構文の学習
  - 変数初期化の際には型宣言が必要
  - 配列を初期化する際の文法：`type array_name = {e1, e2, e3}`
  - 配列.lengthで配列の長さを取得できる
  - 配列の繰り返し処理をするための拡張for文がある
    - Pythonのfor文と近い感覚で書ける　`for(type var：vars)` ≒ `for var in vars`
