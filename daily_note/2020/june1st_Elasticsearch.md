## Elasticsearch
オープンソースの、分散＆スケーラブルな全文検索/分析エンジン。REST APIで操作可能。  
データの保存、検索、分析が可能で、弊社ではログDBとして利用されている。  
### 基本用語
- クラスタ：データ保持を行う1つ以上のノード（サーバ）のコレクションで、ノードを跨いでインデクシング＆検索する  
- ノード：クラスタの構成要素  
- インデックス：RDBでいうDBで、同様の特性を持つDocのコレクションと説明されている  
- タイプ：RDBでいうテーブルで、インデックス内に自由に1つ以上定義できる論理的なカテゴリ  
  - v6以降で非推奨となり、代わりに_docを指定する？  
- ドキュメント：RDBでいうレコードで、インデックス内の情報の基本単位となるJSONデータ  
- シャード＆レプリカ：分散システムとしての要素  
### Search API
基本的に2種類の検索手段が存在する。  
1. `<index>/_search?<queryStrs>`のように、エンドポイントへのGETにクエリパラメータを付与して検索。  
2. `<index>/_search`へ、Query DSLと呼ばれるJSON形式のクエリ文字列をGETのリクエストボディとして送信。（通常のREST APIはGETで送られるボディを想定していないが、RFCで禁じられてはいない。）  
  
基本的なQuery DSL
- `query`：クエリの定義を表し、検索したいドキュメントの条件をここで指定する。  
  - `match_all`：すべてのドキュメントについて一致する。  
  - `match`：あるfieldについて指定した文字列で全文検索（full text queries）。  
    - full text queries：クエリをトークナイズしたものに基づいてfieldを検索して、スコアが高いものが返される（完全一致とは限らない）。  
    - term level queries：クエリをトークナイズせずにfieldを検索する（完全一致）。  
  - `exists`：fieldの有無を条件として検索する。  
  - `bool`：以下の論理演算クエリを組み合わせ、複合的なクエリを定義する。  
    - `must`：AND条件を定義（配列内の全クエリについて真なものを返す）。  
    - `should`：OR条件を定義（いずれかについて真なものを返す）。  
    - `must not`：NOT条件を定義（全クエリについて偽なものを返す）。  
    - これら論理演算クエリの中にboolクエリ自体も定義できる。  
  - `filter`：配下のクエリをFilter contextで実行する。  
    - Query context：「そのドキュメントが他に比べてクエリとどの程度一致しているか（ドキュメントスコア）」という相対的な値を計算して結果を返す。  
    - Filter context：「そのドキュメントがクエリの条件に当てはまるか」というシンプルな条件判定に基づいて結果を返し、スコアに影響は与えない。  
    - テキスト検索以外ではFilter contextがBetter。  
- `aggs`：データの統計情報をグルーピング/展開する。  
  - ヒットしたドキュメントを返すのとは別の結果として扱われる。  
  - `aggs`直下で指定した文字列が集計結果を表すfieldになり、各種クエリを使って集計方法を指定する。  
  - 集計結果は`aggregations`以下に格納される。  
    - その他`took`, `timed_out`, `_shards`, `hits`も`aggs`実行時のリザルト。  
- `size`：そのクエリによって返されるドキュメントの数で、デフォルトは10。  
  - `from`：`size`と併用することで検索結果のページングが可能で、デフォルトは0。  
- `sort`：結果の並び方を指定する。  
- `_source`：ドキュメントについて、この配列で指定したfield（キー）のみを取得する。  

    