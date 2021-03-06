## アルゴリズム  
**Big-O**
- 入力数nに応じてそのアルゴリズムの計算量がどの程度増加するか、という割合を表した記法
- 定数や係数は無視される
- ステップごとに解くべき事象が1/nになっていくアルゴリズムはO(logn)で表される
- **入力数の制約に応じて、最悪どの程度の計算量のアルゴリズムならTLEをくらわないか、という感覚を身につけるのが大事**
  
**安定なソート**
- ソートキーの値が同じ複数の要素を含むデータをソートした時に、処理の前後でそれらの要素の順番が変わらないようなソートのこと
 - IDとスコアをソートキーに持つデータが有ったとして、スコアが同じレコードは常にID順に表示する、など、ソートキーごとに優先度をつければ安定になる？？
 
**挿入ソート**
- 配列を、ソート済みの部分列と未ソートの部分列に分けて扱い、先頭の要素をソート済みとみなして処理をスタートする
- 未ソート部分がなくなるまで以下を繰り返し
  - 未ソートの先頭をinsertingに格納
  - ソート済み部分で、insertingより大きい要素のインデックスを1ずつ後ろにずらす
  - 空いたインデックスにinsertingを格納
- 実装においては、外側のループカウンタで未ソート部分を管理し、内側のループカウンタでソート済み部分を管理する
- 取り出した未ソートの先頭要素より大きいソート済み要素を1つずつ後方にずらしていくので、安定なソートといえる

**バブルソート**
- 以下の処理を、要素数と同じ回数繰り返す
  - 配列の末尾から先頭まで順に隣接要素を比較していき、右側の要素の値の方が小さければ、要素をスワップするのを繰り返す（この処理が1回終わると、ソート済み部分が1つ確定する）
- 実装においては、外側のループカウンタがソート済み部分の個数を表し、内側のループカウンタが、配列の末尾から先頭にかけて比較を行うためのインデックスを表す
- O(N^2)

**選択ソート**：未ソート部分から最小値を選択して、ソート済み部分の末尾に追加する
- 実装においては、外側のループカウンタでソート済みの個数を表し、内側のループカウンタが、未ソート部分の最小値を線形探索するためのインデックスを表す
- O(N^2)
- 離れた要素を交換するので、安定ソートではない

各ソートの特徴
- バブル、選択は、外側のループN回につきN個のソート済み部分が求まり、元データの乱れ具合に依存しない
- 挿入ソートは、外側のループN回につき元データの最初のN個がソートされる＆元データが整列しているほど早くなる

**探索**：データのコレクションの中から、与えられたキーの位置や存在の有無を確かめること
- 線形探索：先頭から順に調査
  - O(N)
- 二分探索：ソート済みコレクションに対して有効
  - 探索範囲の中央の要素を調べる
  - 目的値と一致すれば終了
  - 目的値よりも中央値が大きければ前半部分を、小さければ後半部分を探索範囲として再度中央値を調べる
  - O(logN)
- ハッシュ法：ハッシュ関数を用いて要素を格納する
  - データ格納時に衝突が起きなければ、ほぼO(1)で目的値を取り出せる
  
## データ構造
**Queue**：最初に格納した要素から順に取り出していくFIFOの原則で実装される。別名待ち行列
- 可変長配列と、先頭インデックスのポインタとなる変数だけあれば実装可能だが、要素をDequeueして空いたメモリが無駄になって非効率的
- 循環Queue：メモリのムダを省くために、固定長配列と先頭、末尾インデックスのポインタで実装される
  - Enqueueで先頭に要素を追加して、末尾のインデックスを更新する
  - Dequeueで先頭の要素を削除し、先頭のインデックスをインクリメント
  - **先頭と末尾は、インクリメントしたものを要素数で割った余りで更新できる**
    - **ある限られた範囲で数値をインクリメントし続ける際に汎用的な手法で、必ず0~要素数-1の範囲に収まる**
  - 先頭と末尾のインデックスが同じ時にDequeueするとEmptyになる
  - 末尾+1を要素数で割った余りが先頭と同じになればQueueはFull
    - 実際の要素数をカウントする変数を用意し、それが配列の長さと同一になったときに`isFull`を`True`にするのでもOK
    
**Binary Tree**：木構造の中でも頻出の構造で、親ノードは少なくとも2つの子ノードを持ち、それらの子ノードは何らかの基準で左右に二分される
- 木構造：一つの親要素に対して一つ以上の子要素が紐付けられており、根（頂点の要素）から葉（末端の要素）を辿るにつれて要素が枝分かれして広がっていくような構造
- 以下が要素の取得の手順の代表例で、いずれもStack構造と強く関係
  - Pre-order：root, left, rightの順で走査
    - stackとresultの2つのリストを使う
    - まず起点のnodeをstackに格納して、stackに要素がある間以下の処理をループ
      - resultにnode.valを格納して、stackにnode.right, node.leftの順で格納する
    - resultを返す
  - In-order：left, root, right
    - stack, resultの2つのリストと、現在見ているノードを表すcurrentを使う
    - current or stackに値が入っている間、以下の処理をループ
      - current != nullの間以下の処理をループ
        - stackにcurrentを追加して、currentをcurrent.leftで更新
      - currentがnullになった時、currentをstack.pop()の返り値で更新
        - resultにcurrent.valを追加して、currentをcurrent.rightで更新
  - Post-order：left, right, root
