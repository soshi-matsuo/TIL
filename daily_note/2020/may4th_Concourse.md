## CI/CD
総称して、ビルド/テスト/デプロイの一連部分を自動化してアプリのリリース頻度を高める手法のこと。  
CI/CDの一連のプロセスを称してCI/CDパイプラインと呼ぶ。  
代表的なCI/CDツールがJenkins, CircleCIなどで、Concourseもその一種。  
### CI（継続的インテグレーション）
ビルドやテストを短期間で繰り返し行い開発効率を上げる手法のこと。  
実質的には「こまめにリポジトリにコミットして、その度に自動ビルド・テストを行う」というフローを指す。  
これにより、バグを早期かつ小さいうちに修正できるようになり、開発速度が向上する。  
### CD（継続的デリバリー）
CIを拡張した概念で、成果物を検証環境へ自動デプロイする手法。  
CIの成果物を自動で検証環境に反映させることですぐに本番リリース可能な状態に保ち、リリースまでの時間を短縮する。  
さらに発展して、本番環境デプロイまでを自動化するフローを、明示的に継続的デプロイメントと呼ぶこともある。  
クラウド・コンテナの普及、インフラのコード化によってデプロイの自動化が可能になった。  

## Concouse
Pivotalによって開発されたpipeline-basedなCI/CDツール。Goで開発されている。  
UIでどのJobどうしが依存しているのか、どこで問題が発生しているのかを視覚的に把握できるのが特徴。 
### 独自の概念
**pipeline**：外部の状態を表すresourcesと、resourcesと相互作用するjobを主要素として構成された、CI/CDプロセスを表現するもの。  
- 自己完結的に設計されており、「pipelineさえあればOK」な状況を目指している？  
- YAMLで書かれ、`pipeline.yml`として設定されるのが通例。  
  
**resources**：ソースコードや依存関係、デプロイ状況等、Concourseの外部状態を抽象化したもので、バージョン確認、pull, pushできるようになっている。  
**resource types**：各resourcesに関するメタ情報で、`resouces.type`で指定される。  
**jobs**：実際にパイプライン上で何をするのかを指定する要素。一連のstepで構成され、各jobは冪等かつ疎結合になる。  

### CLI
Concourseは`fly`コマンドを介してCLIで操作を行う。  
- `fly -t <target> login -n <team> -c <concourse-url>`で、指定したConcourseサーバとteamを`target`として登録しながら認証できる。  
  - targetごとにflyのバージョンが指定されており、外れている場合は`fly -t <target> sync`でCLIのバージョンを同期する必要がある。  
  - fly login詳細：https://concourse-ci.org/fly.html#fly-login  
- `fly -t <target> pipelines`でログイン中のteam配下のpipelineを一覧。  
- `fly -t <target> get-pipeline(gp) -p <pipeline>`で特定のpipelineの詳細を閲覧。  
- `fly -t <target> set-pipeline(sp) -c <local_yaml_file> -p <pipeline>`でpipelineの作成/更新。  
- `fly -t <target> pause-pipeline/unpause-pipeline -p <pipeline>`でpipelineの停止/開始。  
- `fly -t <target> destroy-pipeline -p <pipeline>`でpipelineの削除。  

