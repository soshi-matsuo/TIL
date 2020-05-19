## Socket.io
### 概要
クライアント、サーバの双方向、イベントベースのリアルタイム通信を実装するためのライブラリ。  
主にチャットアプリで活用される。
### 実装の流れ
サーバからクライアントに通信する手法としてはemitメソッドが最もポピュラー  
- サーバ上で`io.emit()`すると、イベント名とデータがすべての接続中のsocketに送信される  
- クライアント用のjsファイルの中では、`/*glocal io*/`の下でsocketのインスタンスを生成し、socket.on()でサーバからemitされるイベントをlistenできる  
  
クライアントの切断をlistenするには、サーバ側のconnection listenerの中で、socket(クライアントを表す）に対するdisconnect listenerを実装すればOK  
  
また、socket通信における認証もpassportでカバーされている  
- WebSocketでのチャット通信ではHTTPリクエストが送信されているわけではないので、req.user内のユーザーオブジェクトを検証して認証することができない  
- 接続中のユーザーを確認する方法として、Cookieをパース、デコード、デシリアライズしてユーザーオブジェクトを取得する  
  - CookieもHTTPリクエストに付随するものでは？？  
- socketで認証を行うには、connection listenerを定義する前に`io.use(passportSocketIo.authorize({}))`を呼ぶことで、Socket.ioにオプションを設定する  
  - これにより、connection listenerの中で`socket.request.user`で接続中のユーザーオブジェクトにアクセスできる  
  
クライアント画面に他のクライアントからのメッセージを表示するには、以下のフローを実装する  
- クライアント用のjsファイルで、入力formにsubmitされたテキストを取得し、`socket.emit('chat message', message)`のようにして、サーバへメッセージイベントを発行する  
- サーバのjsファイルでsocket.on('chat message', cb(message))のようにしてクライアントから発行されるメッセージイベントをlistenし、`io.emit('chat message', messageObj)`のようにしてすべてのクライアントにイベントをメッセージ発行する  
- クライアントのjsファイルで`socket.on('chat message', cb(messageObj)`のようにサーバから発行されるイベントをlistenし、そのlistenerの中でチャット画面のDOMを更新する  
  
**要するに、各々のクライアントから送信されるメッセージをサーバで集約し、その他すべてのクライアントの画面に配信して同じように表示するという仕組みでリアルタイムチャットを実現している**  
