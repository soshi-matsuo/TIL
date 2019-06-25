# Java
## 言語仕様
**特殊な型**  
- enum：列挙型とも呼ばれる、クラスの特殊系。定数の集合を保持するために使われることが多い  
- ジェネリクス：総称型とも呼ばれる。一部の特定の型に紐付けられる抽象的な型で、`ArrayList`などの複数の要素を扱うような型において、要素の型を任意の型に限定することが出来る  
  - ex. `List<String> list = new ArrayList<>();`とすることで、要素の型をStringに限定したリストが宣言できる  
  - ジェネリクスを使ったクラスを作成する際は、`public class GenericExample<T>`のようにクラス名に仮型パラメータを設定する  
  
**配列とコレクションフレームワーク**  
- 複数のデータを纏めて扱う仕組みには、大別して配列とコレクションの2種類がある  
- 配列：インデックスで要素を指定する固定長配列で、要素数や内容が定まっている場合に使われる  
- コレクション：可変長で、任意の数の要素を格納できる。List, Map, Setなどが代表的なインターフェースとして存在する（各々異なる実装を持つ）  
  - 繰り返し処理の際に、コレクションをそのままfor文にわたすだけではなく、コレクションからIteratorインターフェースを生成したものをfor, while文に渡して処理を行うことで、例えばコレクション内の要素の削除など（Iteratorが持つメソッドを使って）個々の要素に対する操作を行いながら繰り返し処理が出来る  
  - Mapのキーの性質とSetの要素の性質は一意性制約の点で共通しており、この性質を活かして、Setの内部実装にはMapが使われている。（Setに追加される要素は内部的にはMapのキーとして保持される）  
  
**例外処理**
- 検査例外（Exception）：プログラム作成時に想定可能な例外。APIの通常の使い方で発生しうるようなものが該当する？  
  - `catch`ブロックで補足するか、そのメソッドの呼び出し元に対してthrowすることが求められる
- 実行時例外（RuntimeException）：想定外の例外。ゼロ除算や型エラーなど、そのAPIが想定していない使い方をすると発生する  
  - プログラムで補足する必要はない。無条件で例外発生コードの呼び出し元で発生する
- エラー：システム動作を停止すべき致命的なエラー（スタックオーバーフローなど）  
- 処理の基本として、「スタックトレースをログに出力する」「テキトーに`throw Exception`を書かない（弊害が大きい）」「処理を中断するか、継続するならデフォルト値を設定する」などのBPがある