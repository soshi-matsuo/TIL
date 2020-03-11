# Node.js
## helmetJS
### セキュリティに関わるヘッダ
- X-Powered-By：サーバで稼働しているプログラムを表すレスポンスヘッダ  
  - PHP, Expressなどは自動でこれを返すが、セキュリティの観点からは隠すべき  
- Content Security Policy（CSP）：そのページがリソースを取得できるWebページのホワイトリストを表すレスポンスヘッダ  
### helmetJS
- Expressを使ったWebサイトのセキュリティ設定が楽に出来る  
- 以下の9つのMiddlewareで構成され、app.use(helmet())でnoCacheとCSP以外はまとめてマウント可能  
  - `hidePoweredBy`
  - `frameguard`
  - `xssFilter`
  - `noSniff`
  - `ieNoOpen`
  - `hsts`
  - `dnsPrefetchControl`
  - `noCache`
  - `contentSecurityPolicy`
## bcrypt
- パスワードハッシュ化のアルゴリズムで、salt（ランダムデータ）の付与、ストレッチング（複数回ハッシュ関数を適用）などを備えている  
- Node.jsではそのまま`bcrypt`としてnpmで提供されている  
  - データのハッシュ化は計算コストが高いので非同期で実行すべき  
  - `bcrypt.hash(pw, saltRounds, callback)`でハッシュ化
    - CB関数内ではDBへの保存や`bcrypt.compare(plain, hash, callback)`による平文との比較を行う  
  - `bcrypt.hashSync(pw, saltRounds)`で同期実行も可能  
## Chai
- ユニットテストのライブラリ  
- `assert.deepEqual(obj1, obj2, message)`：2つのオブジェクトのプロパティまで再帰的に探索し、==で比較する（===で比較したい場合は`deepStrictEqual()`を使う）  
- `assert.isAbove(n1, n2, message)`：n1 > n2のときにtrueを返すアサーションで、isAtMost(n1, n2)はn1 <= n2のときにtrueを返す  
  - `isBelow()`, `isAtLeast()`は逆の関係を検証する  
- `assert.approximately(actual, expected, delta)`：actualがexpectedの+-deltaの範囲の値であるかを検証  
- `assert.match(str, regex)`：正規表現マッチの検証  
- `assert.property(obj, prop)`：プロパティの存在の検証  
- `assert.typeOf(obj, str)`：objがstrで表される型であることを検証  
  - オブジェクト型に対して、typeof演算子の結果より詳細に確認可能  
- `chai-http`プラグインを利用することで、APIエンドポイントの機能に対してもテスト出来る  
