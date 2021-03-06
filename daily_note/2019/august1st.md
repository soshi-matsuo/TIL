# Python
# Head First
- 内包表記
  - 既存のコレクションを処理して新しいコレクションを作るときの「空のリスト・辞書・集合を作成し、forで既存のコレクションを処理しながら、作成済みの空のコレクションに要素を追加」という共通のパターンを簡略化したもの
  - タプルはイミュータブルで、一度生成されたら要素を変更できないので、他のデータ構造と違って内包表記で生成することはできない
  - リスト内包表記の`[]`を`()`に置き換えると、ジェネレータが作られる
- ジェネレータ
  - 通常のリスト内包表記とジェネレータは最終的に同じデータを持つが、実際にデータが生成されるタイミングが異なる
  - 内包表記だと、それが宣言された箇所でリスト内のすべての要素が一度に生成されるので、巨大なデータを扱う際は「リストの生成部分がblocking codeになる」「処理が完了するまでメモリを圧迫する」などの問題が起こりうる
  - 一方ジェネレータが一度にメモリに保持するデータは1つだけで、ジェネレータが生成したデータを処理するコードもすぐに処理を開始できる
  - 関数内にジェネレータを埋め込むことも可能で、このようなジェネレータ関数に値を返させるためにはreturnではなくyieldを使う。これによって、任意のイテレータから呼び出し可能なジェネレータ関数が定義され、以下のように処理が進む
    - ジェネレータ関数がイテレータから呼ばれると、ジェネレータ関数はまず1つの返り値をyieldで呼び出し元のイテレータに渡し、待機状態に入る
    - 呼び出し元では返された値の処理を行い、それが完了すると再びジェネレータ関数を呼び出す
    - これらのプロセスが、ジェネレータ関数に渡された要素が無くなるまで繰り返される

## Flask Web Development
### Chapter2
- WSGI： Web Server Gateway Interfaceの略で、サーバがFlaskオブジェクトにクライアントからのリクエストを渡すためのプロトコル
- `__name__()`をコンストラクタに渡すのはFlaskにアプリのメインモジュールを教えるため。これによって、Flaskがその他のモジュールやテンプレートのディレクトリ位置を見つけられるようになる
- アプリ内の処理とURLの対応付け（ルーティング）はデコレータで行う
  - `@app.route()`によるルーティングはview用の関数に使われることが多い
  - `app.add_url_rule(url, endpoint, function)`を使って明示的にルーティングを行う事もできる
  - routingの際にurlの一部を<>で囲むと、その部分の値を動的なものとして扱うことができ、view関数の引数に渡すことも出来る。/user/<int:id>のように型指定も可能
  - view関数：アプリのURLを処理するための関数で、これの返り値はクライアントが受け取るレスポンスになる
- 環境変数`FLASK_APP`にappを初期化しているPythonファイルの名前を設定した上で`flask run`コマンドを実行すると、localhost:5000でflaskのWebアプリが起動する。デバッグモードは`FLASK_DEBUG=1`で設定可能
  - スクリプト上では`app.run(debug=True)`で同じように起動出来る
- `request`などのいくつかのコンテキスト変数は、importしておくことで、各view関数から（スレッドセーフで）グローバル変数のようにアクセスできる。これはFlaskのContext機能のおかげで、Flaskには大別してapplication context, request contextの2種類が用意されている
- `make_response()`でレスポンスオブジェクトを生成する。これを使うことで、view関数内で返すべきレスポンスのヘッダー、ボディの情報を操作することが出来る
  - `redirect(url)`をview関数の返り値にセットすることでリダイレクトが実装可能（デコレータでも出来る）
  - `abort()`でHTTPにまつわる例外をraiseすることができる
### Chapter3
- Flaskにおけるview関数は、ビジネスロジックとプレゼンテーションロジックの2つの役割を持ち、これらは設計上独立して定義されるべき
  - プレゼンテーションロジックを切り出したのがテンプレートで、テンプレート内の一時変数を実際の値に置換して実際にレスポンスとして返される形に描画するのがテンプレートエンジン
- `render_template(template_file, placeholder=actual_value)`で、template_file内の一時変数に値を渡しながらテンプレートをレンダリング出来る
  - テンプレートの一時変数の定義では、パイプでつなげてcapitalizeなどのフィルターを指定することが出来る
    - ex. `{{ some_value | safe }}`で、some_valueをエスケープせずにレンダリングすることができる
  - if, for, 継承などの制御構文は{% %}で明示される
  - baseから継承したblock内でbaseの要素をそのまま使いたい場合、`{{ super() }}`と書けばOK
- Bootstrapと連携する拡張機能がある（要モジュールインストール）
