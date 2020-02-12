# JavaScript
## Stream API（特にReadableについて）
- Stream API：Node.js上でデータストリームを扱うインターフェースのこと  
- ストリーム：準備のできたデータから順に、連続的に処理するようなI/Oの形式のこと  
  - 通常のI/O処理では、ファイル全体を一度に読み込むことでメモリを圧迫するので、サイズが大きい場合はストリームでI/Oを行うべき  
  - 動画ストリーミングのストリームもここから来ている  
- Streamでデータを取得するには、Streamインスタンスを生成した後、フローモード、ポーズモードのいずれかの手段をとる  
  - フローモード：イベントの発生に伴い、Streamからそのイベントへ自動的にデータが送られる  
    - `readable.on('data', handler)`のように、dataイベントにイベントハンドラを登録することで、ハンドラの中で入力データを扱える  
    - `readable.on('end', handler)`で読み込みの終了を検知出来る  
    - `readable.on('error', handler)`で例外処理が可能  
  - ポーズモード：Streamに読み込まれるデータはプログラムから能動的に取得される  
    - `readable.on('readable', handler)`でStream内に取得可能データが流されたことを検知できる  
    - `readable.read(numOfByte)`で、Stream内の未取得データを取得する  
      - 未取得データが存在しない、データ取得が完了済み、データが1バイト未満の場合はnullが返される  
    - readableイベントハンドラの中で`read()`を呼び出すことで、取得可能になったデータを逐次的＆能動的に読み込む、という流れ
- pipeについて
  - Readable StreamをWritable Streamに接続するための機能  
  - `readable.pipe(writable)`の形で書き込みStreamに接続することで、読み込みStreamからデータを取得することが出来る  
    - 例えば、csv-parseから生成されるparserは、Readable, Writableの性質を兼ね備えたDuplexなStream  
    - よって、`fs.createReadStream().pipe(csvParser)`で、fsのStreamに読み込まれたデータをそのままパーサのStreamに渡せる  
## Map
### 基本仕様
- JSのMapは連想配列を表現するための型で、`const map = new Map()`のように、コンストラクタで初期化を行う必要のあるオブジェクト  
- Mapの要素はエントリーと呼ばれ、エントリー自体は`[key, value]`の形を取る配列  
- 要素数は`size`プロパティで取得可能  
- `Map.get(key)`で指定したKeyに紐づくValueを取得でき、`Map.set(key, value)`でエントリーを追加できる  
- `Map.delete(key)`で指定したKeyを持つエントリーを削除でき、`Map.clear()`で全てのエントリーを削除できる  
### データ構造としてのObjectとの対比
**基本的には、純粋にKeyValue形式のデータ構造を表現したい際は、ObjectよりもMapを使うべき**らしい  
- Objectはプロパティとしてデータを保持するため、連想配列として扱う際に以下の問題がある  
  - `Object.prototype`のプロパティと保存したいデータのKeyが衝突する可能性がある  
  - Keyとして使えるデータ型に制約がある  
- Mapのデータ保持の仕組みはプロパティとは異なるので、上記問題が解決されている  
- 実際は、以下の利点を考慮してケースバイケースでデータ型を定める  
  - Objectに対する、連想配列としてのMapの利便性
    - Iterableなので、for of文で簡単に反復処理出来る  
    - 要素数もObjectより楽に取得できる  
  - 従来どおり、Objectを連想配列として扱う利点  
    - リテラルで定義しやすい  
    - JSON変換が簡単  
    - BIFや外部ライブラリで連想配列として渡されることが多い  
