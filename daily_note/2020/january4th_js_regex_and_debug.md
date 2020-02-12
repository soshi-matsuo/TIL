# JavaScript
## Regex
### 正規表現リテラル
- データ型としてはオブジェクト（参照型）  
- `/regex/`のように、JSではスラッシュで囲むことで正規表現リテラルを定義できる  
  - `/regex/i`のようにスラッシュの後ろに特定の文字を渡すことで正規表現フラグを使える  
    - `/regex/i`：ignore caseで、大文字小文字を無視してマッチする  
    - `/regex/g`：global matchで、全てのマッチする部分を探す  
  - `.`でのワイルドカードや、`[]`での文字クラスの表現、`(?=)`などの先読み返り読み系もそのまま表現可能  
### メソッド
- regex.test(str): bool`で、文字列が正規表現にマッチするか否かを検証できる  
- `str.match(regex): array`で、引数の正規表現にマッチする最初の部分文字列を配列で取得できる  
  - マッチする部分を全て取得するにはglobalフラグを使う  
## デバッグTips
- デバッグの基本は変数の中間値や型を確認すること  
- console.log(), typeofがJSデバッグの基本  
- console.clear()を実行することで、ブラウザのConsoleログをクリア可能  
### よくあるバグや、バグの原因  
#### 一般的ミス  
- スペルミスによるReferenceError  
- カッコの数のズレ（エディタが補正はしてくれるが、既存データを修正したりCB関数を扱ったりする際には要注意）  
- quotingの混在による文字列のSyntaxError  
- falsyな値はfalse, 0, "", NaN, undefined, nullの6つ（空配列や空オブジェクトはture！）  
#### 関数
- 関数オブジェクトの参照と、関数の返り値の参照を間違える（()の有無で値が変わる）  
- 関数へ与える引数の順序ミス  
- 返り値が、Booleanなのか、処理を行った結果の値なのか、処理に使った値なのかを混同する  
#### 反復処理
- Index out of range  
- 情報を累積する変数を初期化してしまう/初期化すべき変数をそのままにしてしまう  
- 終了条件とインクリメント、デクリメントの食い違いによる無限ループ  