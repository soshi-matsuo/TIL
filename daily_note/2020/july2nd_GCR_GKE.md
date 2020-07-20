## 概要
**GCR**：Google Container Registryの略で、Google Cloud Platform上で実行される非公開のコンテナイメージレジストリのこと。  
  
**GKE**：Google Kubernetes Engineの略で、GCP上に構成されるコンテナ化されたアプリケーションのオーケストレーション（デプロイ/管理/スケーリング）を行う環境のこと。  
GKEのk8sクラスタは、Google Compute Engine上の複数のVMインスタンスをグループ化したものから構成される。  

## ConcourseからGCRへのデプロイ手順  
1. GCRへアクセスするためのサービスアカウントを作成してjson_keyを生成  
2. VaultなどのCredential Managerにjson_keyを保存  
3. registry-image ResourceをGCRへ紐付けて、Concourseパイプラインを定義  
4. `task`でDockerイメージをビルドして、3で定義したResourceへ`put`する  
### Tips
#### サービスアカウント
- GCP上のアプリケーションに外からアクセスする権限を自分のアプリケーションに与えるには、サービスアカウントを利用できる  
  - 公式：https://cloud.google.com/iam/docs/service-accounts?hl=ja
- 一定の操作に専念するサービスアカウントを作成する  
  - このケースだと、Container Registoty用だと分かるようにネーミングする  
  - 作成しただけでは何のRoleも付与されないので、IAMでの一覧には出てこない  
- 作成したサービスアカウントに適切なRoleを付与する  
  - IAMタブ上部の追加ボタンで、サービスアカウントにそのProjectでのRoleを付与する  
  - GCRを操作したければ、`Strage.Admin`のRoleを与える  
    - GCPにおいて、GCRとやり取りするための権限が明示的に存在するわけではない  
    - GCR操作の権限付与：https://cloud.google.com/container-registry/docs/access-control  
- 適切な権限を付与したあと、メニューから鍵の生成を選択してJSONキーをDLする  
  - キー生成：https://cloud.google.com/container-registry/docs/advanced-authentication#json-key
#### GCRとGCSの関係
- GCRは、自身の属するProjectのGCSバケットにコンテナイメージを格納する  
- GCRへImageをPushした際に、`[REGION].artifacts.[PROJECT-ID].appspot.com`の命名規則で、自動的にバケットが作成される  
- GCRのイメージは、内部的にはGCSバケット内のオブジェクトとして管理される  
- GCRへアクセスするには、そのregistryが属するバケットに対する権限を与える必要がある  
- Image読み書きを許可するには、バケットに対する`StorageAdmin`権限を与える  

## ConcourseからGKEクラスタを操作する手順
1. Kubernetes Engine developerの権限を付与したサービスアカウントを作成し、json_keyを生成  
2. `kubectl`を実行するJobの`task.params`へ`GOOGLE_APPLICATION_CREDENTIALS`を宣言し、1のjson_keyへのパスを値として格納する  
3. 操作したいGKEクラスタの情報を`gcloud`を使って取得し、それを元に`kubeconfig`ファイルを生成して、2の実行環境で`KUBECONFIG`として定義する  
4. Jobから`kubectl`を介してGKEを操作  
### Tips
- **あらゆるGKEクラスタは、`kubectl`から渡された認証情報を検証して外部アクセスを受け入れる**  
- k8sの認証には、一般的にユーザーアカウントまたはサービスアカウントが必要  
  - ユーザーアカウント：k8s内で認識されるが、k8sよりもメタな存在。作成や管理は外部で行われる  
    - **GKEクラスタにおいては、GoogleアカウントまたはGCPサービスアカウントがk8sユーザアカウントとなる**  
  - サービスアカウント：k8sによって作成、管理が行われるが、Pod等k8sによって作成されるオブジェクトでしか使用できない    
    - k8sのサービスアカウントとGCPのサービスアカウントは別の概念  
    - [zlabjp/kubernetes-resource](https://github.com/zlabjp/kubernetes-resource)を利用する場合はこちらを使う  
- `kubectl`は実行時、クラスタ情報等が含まれたconfigファイルを以下の優先順位で探索する  
  - `--kubeconfig`フラグで与えられるファイルパス  
  - 環境変数`$KUBECONFIG`で定義されるファイルパス  
  - `$HOME/.kube/config`  
- `kubectl`が実際にGKEクラスタへ認証される仕組み
  - `kubectl`は、実行されると上述の通りkubeconfigファイルを自動的に読み込む  
  - kubeconfigファイルの`users.user.auth-provider.name`の値が`gcp`に設定されていた場合、`kubectl`内部のgcp auth pluginが`$GOOGLE_APPLICATION_CREDENTIALS`を探して自動的に読み込む  
  - 認証がうまくいくと、寿命1時間のアクセストークンがkubeconfigの`users.user.auth-provider.config`へ書き込まれる  
- `KUBECONFIG`, `GOOGLE_APPLICATION_CREDENTIALS`ともに、**その情報を格納したYAML/JSONファイルへのパスを指定しなければならない**  
- クラスタ情報と認証情報の指定が正しいのに接続できない場合、GKEクラスタが属するVMインスタンスへファイアウォールが設定されている可能性があるので注意  



