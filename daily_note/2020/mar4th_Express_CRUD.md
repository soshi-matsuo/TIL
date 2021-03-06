# ExpressによるCRUDなWebアプリ
## テンプレートエンジン in Express
WebアプリにおいてViewページをかんたんに実装するために使われるのがテンプレートファイルで、HTMLの構造をより簡潔に書き表すことができ、文書中にサーバから値を受け取るための変数(placeholder)を利用できる。  
  
テンプレートエンジンは、レンダリングの直前に、テンプレートファイル内の変数をサーバから受け取った値に置換して静的HTMLファイルに変換する。  
  
具体的には以下の手順でExpressから利用できる。  
- `npm install`で任意のエンジンを依存関係に追加。  
- ルーティングファイルで`app.set('view engine', <templateEngine>)`を呼ぶことで、view engineというconfig値に利用したいエンジンを設定可能。  
- テンプレートファイルが置かれるディレクトリを`views`と呼び、`app.set('views', <viewsDirsPath>)`で設定可能。  
- ルーティング関数内で`res.render(<viewPage>, <valueObj>)`を呼ぶことで、テンプレートファイルに値を渡す。  

## mysql in Express
- `npm install mysql`でMySQLのドライバをインストール。  
- MySQLを起動し、`mysql.createConnection({ host: <hostName>, user: <userName>, password: "password" })`でconnectionオブジェクトを初期化。  
- `connection.query(<query string>, <valueArr>, CB)`という構文でMySQLのクエリを実行でき、実行結果をCB関数で受け取れる。  
  - query string内で`?`(placeholder)を使うと第二引数の配列の先頭から順に値を受け取れる。  

## POSTリクエストの基本
### フォームによるPOST
HTMLの`<form action="<path>" method="POST">`を介してPOSTリクエストが実行されると、HTTPリクエストヘッダの`Content-Type`が`application/x-www-form-urlencoded`に設定される。  
  
このContent-Typeは、リクエストボディの値がURLエンコーディングされていることを表し、ボディをサーバで扱うためにはパース（デコード）する必要がある。
  
Express4.16以降では、従来`body-parser`として提供されていた上記を実現する機能がデフォルトで実装されており、ルーティングファイルで`app.use(express.urlencoded({ extended: true }))`すればリクエストボディを扱える。  
### ブラウザリロードによる意図しない再POST
前提として、**ブラウザの更新機能は直前のHTTPリクエストを再度実行する**機能である。  
なので、POSTリクエストを送信した直後に更新ボタンを押すと同じPOSTリクエストが再び実行される。  
これにより、直前にDBへの追加処理などを行っていた場合、更新するたびにデータが追加されていくような挙動になってしまう。  
  
このような挙動を防ぐために、通常はPOSTのエンドポイントの処理の最後に、クライアントへ別のページへのアクセスを要請する（リダイレクトする）。  
- リダイレクト：そのレスポンスを受け取ったクライアントに、指定したURLへのリクエストを自動的に送らせる、というようなレスポンス。  
  
これによって、**クライアントから見た直前のリクエストは、リダイレクトで指定されたページへのGETリクエストになる**ので、ReloadだけでPOSTが実行されてデータが追加されるような挙動を防ぐことが出来る。  
### 安全性と冪等性
HTTPメソッドは、その**安全性**と**冪等性**において区別される事が多い。POSTは安全でも冪等でもない。  
- 冪等性：あるリソースに対して、同じリクエストを何度実行しても同じ結果（同じリソースの状態？）が得られる、という性質。  
- 安全性：リクエストの対象にとったリソースに、そのリクエストの実行によって変化を与えないという性質。  
  
POSTでリソースの更新や削除を行うことも可能だが、PUTやDELETEと違って冪等ではないので注意が必要。  
## Webアプリにおける認証
現代のWebアプリの殆どでは、クライアントはリクエストの都度認証を行うのではなく、最初のログインリクエストでのみ認証情報をサーバに送信し、その後はセッションを介して認証状況を管理している。  
具体的には、おおまかに以下のような流れで認証情報をやり取りする。  
- 初回リクエストを受け、サーバがセッションIDを発行＆セッションデータを作成。  
- クライアントはセッションIDをCookieとして保存し、次回以降のリクエストに付与する。  
- ログインページから、クライアントが認証情報を送りログイン成功。  
- クライアントのCookie内のセッションIDと紐づくセッションデータに、「そのクライアントが認証済みである」という情報が記録される。  
- セッションが有効な間は、CookieでセッションIDをサーバに伝えるだけで、そのクライアントが認証済みだと示すことが出来る。つまり、そのセッションの間ログイン状態を維持できる。  
### Expressの認証機能
主に以下を利用して実装する。  
- Passport：ユーザーの認証を実装するためのnpmパッケージ  
- express-session：Express上でセッションを扱うためのミドルウェア  
  
セッション管理をExpressで実行するには以下の手順を踏む  
- express-session, passportをインポートし、以下の３つを`app.use()`でミドルウェアとして登録  
- `session({ secret: process.env.SESSION_SECRET, resave: true, saveUninitialized: true })`  
- `passport.initialize()`  
- `passport.session()`  
  
実際にPassportでセッション管理を行うには、以下のシリアライズ、デシリアライズの手順が必要。  
- 参考：https://stackoverflow.com/questions/27637609/understanding-passport-serialize-deserialize  
- `passport.serializeUser(cb(userInfo, done))`  
  - passportでの認証が成功した際に一度だけ呼ばれ、そのUserの情報をシリアライズする  
    - 慣例的には、UserIDのみがシリアライズ対象とされる  
  - CB内の`done(null, userInfo)`の呼び出しによって、シリアライズされたUser情報はセッションデータとしてサーバに保持される  
    - 具体的には`req.session.passport.user`に保持される  
- `passport.deserializeUser(cb(userInfo, done))`  
  - 認証成功後のリクエストのたびに呼ばれる  
  - cookie内のセッションIDからセッションデータを特定し、そこに保持されたシリアライズ済みのUser情報を、cbのuserInfoに引数として渡す  
  - cb内では以下の処理が走る  
    - userInfoを使ってDB等からUserオブジェクトを検索＆取得  
    - `done(null, userObj)`の呼び出しによって、取得したUserオブジェクトをそのリクエストの`req.user`に格納  
  
上記2つのメソッドによって、リクエストを扱う際に`req.user`を検証すれば認証済みか否かがわかる。  
- 認証済みでなければセッションデータにシリアライズ済みUser情報が入っていないので、`deserializeUser`によって`req.user`にUserオブジェクトが格納されることもない  
- 実際に各ページで認証の有無を検証するには、`req.isAuthenticated()`（passportを初期化すると自動で実装される？）を使ってミドルウェアを作成し、ルーティングに登録するのが慣例  
  
具体的にユーザー認証を実装するには、PassportのStrategyオブジェクトを活用する。StrategyではGoogleやGithubのソーシャル認証など様々な認証フローを選択可能。  
  
以下`LocalStrategy`(アプリ内でDB等のデータソースを使って認証)での例  
- `passport.use(new LocalStrategy())`のようにして認証手法を定義する  
- LocalStrategyでid, password認証を行う場合は、`new LocalStrategy(cb(username, password, done))`を`passport.use`に渡して、cb内で具体的な認証手順を実装する  
- リクエストに対してStrategyで設定した認証を実際に行うには、ログイン用のPOSTエンドポイントのルーティング関数に、引数として`passport.use('local', { failureRedirect: '/' })`を渡してミドルウェア登録をすればOK  
  
ユーザー登録を`LocalStrategy`で実装するには、以下の3ステップを個々のミドルウェアとして実装する（要パスワードハッシュ化）。  
- DBへのユーザーオブジェクトの保存  
- 保存されたユーザーについての認証  
- 認証済みページへのリダイレクト  
  
  
Passportでは**ソーシャル認証（OAuthに則った、外部サービス内のユーザー情報を利用しての登録・認証）も可能**で、以下が典型的な流れ。  
- ユーザーがソーシャル認証用のエンドポイントにルーティングされたリンクをクリック  
- ルーティング関数がGithubなどのStrategyを呼び出し、ユーザーをリダイレクトする  
- 未認証の場合、外部サービスがログインを要求し、そのサービス上のユーザーデータへのアクセス権をアプリに付与するようにユーザーに要求  
- アクセス権を付与することに同意した場合、ユーザーは、特定のコールバックを経由して、サイト上のユーザーデータを伴ってアプリに戻る  
- アプリ側でユーザーデータを検証し、認証済みか否かを判断する
  
OAuthなので、リソースサーバ（外部サービス）からClient_id, Client_secretを取得して`.env`ファイルなどに保管しておく必要がある。  
また、GitHubStrategyなどの`OAuth2Strategy`をpassportに登録する際は、credentials, callbackURLを含んだオブジェクトと、外部サービス上でユーザーの認証が成功したときに呼び出されるCB関数の2つを引数に渡す必要がある。  

## Expressサーバのモジュール化
Expressサーバにおいても、例えば認証系の処理とルーティング系の処理のように、司る機能ごとにファイルを分離して管理したほうが保守性が高まる。  
FCCの例では、ルーティングをすべて`routes.js`に、認証関係をすべて`auth.js`にまとめて切り出しており、いずれも`module.exports = (app, db) => {}`とすることで、app（Expressのインスタンス）, db（MongoDBのインスタンス）を受け取る無名関数としてモジュール化している。  
また、切り出したモジュールが依存しているライブラリ/モジュールがある場合は、そのモジュールのjsファイル内でもrequireで読み込む必要がある。  
利用する際には、サーバを立てているjsファイル上で、モジュール化した無名関数を`require`して実行すればOK。  
  

