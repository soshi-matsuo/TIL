# Okta OpenID Connect & OAuth2.0 API
**OAuth2.0やOpenID ConnectのフローにおけるAuthorization Serverの役割を担えるクラウドサービス**
- 公式：https://developer.okta.com/docs/reference/api/oidc/#endpoints

## OAuth2.0
### 概要
あるクライアントアプリに対して、リソースサーバ上のデータを利用するのを限定的に認可するためのフロー、仕様。  
RFC6749で定義されている。  
アクセストークンを発行するまでの手順に関する仕様とも言える。  
フローは以下の5種類に大別され、authorization code flow（認可コードフロー）が最もポピュラー。  
  - authorization code flow
  - implicit flow
  - resource owner password credentials flow
  - client credentials flow
  - refresh token flow  
  
### 認可コードフロー
主にauthorization_codeの発行、アクセストークンの発行、リソースの取得の３段階に分けることが出来る
1. 前もって、リソースサーバ上にクライアントアプリと求めるスコープを登録して、クライアントIDを発行しておく  
2. エンドユーザーがクライアントアプリ上で、リソースサーバ上のデータへのアクセスを要求  
3. クライアントアプリはエンドユーザーに対して、まずリソースサーバ上での認証を要求  
4. エンドユーザーは認証情報をリソースサーバに渡し、authorization_codeをサーバから取得  
    - サーバ上のauthorization_code発行を担うAPIを **認可エンドポイント（/authorize）** と呼ぶ  
5. エンドユーザーはクライアントアプリにauthorization_codeを預ける  
6. クライアントアプリは、自らが登録済であることを示すクライアントIDと、リソースサーバからの認可が下りている証であるauthorization_codeを、リソースサーバに提示  
7. リソースサーバはクライアントアプリに対して、「エンドユーザー所有のデータを、限られたスコープにおいて操作できる権利」としてのアクセストークンを発行  
    - サーバ上のアクセストークン発行を担うAPIを **トークンエンドポイント（/token）** と呼ぶ  
8. クライアントアプリは、アクセストークンが有効な間、エンドユーザーの代わりにリソースサーバの該当データに繰り返しアクセスできる  
  
上記は簡略化したもので、実際はアクセストークン発行までのフローは認可サーバが担い、リソースサーバはアクセストークンを元にデータを提供するだけ  
参考  
- https://qiita.com/TakahikoKawasaki/items/200951e5b5929f840a1f  
- https://tools.ietf.org/html/rfc6749
  
## OpenID Connect  
### 概要
OAuth2.0を拡張する形で定められた仕様で、OAuth2.0がアクセストークン発行の仕様（**≒認可の仕様**）である一方、OpenID ConnectはIDトークン発行の仕様（**≒認証の仕様**）だと言える。  
公式曰く、OpenID Connect1.0 = OAuth2.0 + (ID, 認証) とも説明される。 
  
参考：https://qiita.com/TakahikoKawasaki/items/498ca08bbfcc341691fe#_reference-9686d2988451466463eb  

### フロー概略
1. クライアントアプリがOpenIDプロバイダに対してIDトークンを要求
2. OpenIDプロバイダが、クライアントアプリに対してIDトークンを発行してよいかを、エンドユーザーに確認
3. IDトークンの発行に同意する場合、エンドユーザーはOpenIDプロバイダに対して認証情報を提示
4. 認証情報の検証に成功した場合、OpenIDプロバイダはIDトークンを生成し、クライアントアプリに対して発行  
  
OpenID ConnectはOAuth2.0と酷似したプロセスが定められているため、**OpenID ConnectにおけるOpenIDプロバイダはOAuth2.0における認可サーバを兼ねることが大半**。  
つまりクライアントアプリは、アクセストークンとIDトークンを同時に要求することが可能。
  
参考  
- https://qiita.com/TakahikoKawasaki/items/4ee9b55db9f7ef352b47  
- http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html  

### IDトークン
サービス間ID連携のために利用されている概念（またはデータ構造）。  
  - 一度あるサービスで認証を行ってOpenIDプロバイダからIDトークンが発行されれば、有効期限の間、連携している他のサービスでの認証を省ける、という意味でのID連携  
  
ユーザーがあるOpenIDプロバイダにおいて認証済みである証明と、そのユーザーの属性情報が含まれているという性質を持つ。  
また、電子署名が必須となっているため捏造を予防できる。  
  
実際のIDトークン発行手順は、**OAuth2.0の認可エンドポイントの仕様を拡張**することで定められている。  
- OAuth2.0の仕様では認可エンドポイントのリクエストパラメータ`response_type`は`code`または`token`のみを受け付けていた
  - OpenID Connectでは受付可能な値として`id_token`, `none`が追加され、`code`, `token`, `id_token`の任意の組み合わせを受け付けるように変更された
- IDトークン発行のリクエストは`scope`に`openid`を含む必要がある
  - `scope`パラメータを使うことで、IDトークンのclaimに含める属性情報を追加することも出来る  
    - profile, email, address, phoneなどは標準で要求可能になっている
- `response_type`が`code`で`scope`に`openid`が含まれるパターンが多い
  - 認可エンドポイントから認可コードが発行されるまではOAuth2.0と同じだが、そのコードを使ってトークンエンドポイントにリクエストを送ると、アクセストークンに加えてIDトークンが返される
  - `scope`による指定がない場合は単にアクセストークンが返される  
    
IDトークンそのものは一定の構造からなる文字列データで、JWS, JWE, JWTというデータ構造と密接不可分。  
最もシンプルなものは`ヘッダ.ペイロード（本文）.署名`という構造で、これはJWSのCompact Serialization形式。  
`ヘッダ.キー.初期ベクター.暗号文.認証タグ`という5フィールドの構造を持つ場合もあり、これはJWEのCompact Serialization形式。  
- **JWS：JSON Web Signatureの略で、JSONベースのデータ構造を用いて、電子署名などで保護されたデータを表現する形式**  
- **JWE：JSON Web Encryptionの略で、JSONベースのデータ構造を用いて、暗号化されたデータを表現する形式**  
  - IDトークンの文脈でJWEが使われる場合、暗号文フィールドにはJWSを暗号化したものが入る  
  - JWEでは平文の暗号化→その暗号化に使った鍵の暗号化、という風に2段階の暗号処理が行われている  
    
JWS, JWEの各部はそれぞれ元のデータをbase64urlエンコードしたもの。  
- base64urlエンコード：base64エンコードをurlセーフ（URL上で使える形）に修正したもの
  - base64エンコード：マルチバイト文字を含むすべてのデータを[a-zA-Z0-9], +, /の64文字で表す
    - データ長の調整（パティング）に=を使うので、実際は65種類の文字
    - 昔のメールプロトコルSMTPでは英数字しか送れなかったので、すべてのデータを英数字のみで表すMIME規格が登場し、その中でbase64が定められた
    - URL上では+, /などは予約語になるので、URLセーフな拡張形が必要とされ、base64urlへと発展
    - 参考：https://qiita.com/PlanetMeron/items/2905e2d0aa7fe46a36d4  
    
より狭義には、IDトークンの構造はJWTと呼ばれる構造をとっている。  
- **JWT：JSON Web Tokenの略で、「JSON形式で表現されたclaimの集合を、JWSまたはJWTに埋め込んだもの」と定義される**  
  - claim：JSONオブジェクト内のkey:valueペア部分のこと
    
IDトークンは上記の通り、JWSまたはJWEの構造になっていると言えるが、それらのペイロードは、トークン発行者や宛先などの必要な情報をclaimの集合として表すJSONである。  
よって、IDトークン＝JWT構造と言える。  
JWTには、「claim集合をペイロードに持つJWS/JWE」形式、「claim集合をペイロードに持つJWS/JWE自体をペイロードとしたJWS/JWE」形式があるが、**IDトークンとしてJWTを作る場合、「署名が必須で、暗号化する場合は署名→暗号化の順番で行う」と定められている**ので、JWS形式または、JWSをペイロードに持つJWE形式の2パターンのいずれかになる。  
  
参考：https://qiita.com/TakahikoKawasaki/items/8f0e422c7edd2d220e06
