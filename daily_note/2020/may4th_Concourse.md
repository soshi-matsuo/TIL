# CI/CD
総称して、ビルド/テスト/デプロイの一連部分を自動化してアプリのリリース頻度を高める手法のこと。  
CI/CDの一連のプロセスを称してCI/CDパイプラインと呼ぶ。  
代表的なCI/CDツールがJenkins, CircleCIなどで、Concourseもその一種。  
## CI（継続的インテグレーション）
ビルドやテストを短期間で繰り返し行い開発効率を上げる手法のこと。  
実質的には「こまめにリポジトリにコミットして、その度に自動ビルド・テストを行う」というフローを指す。  
これにより、バグを早期かつ小さいうちに修正できるようになり、開発速度が向上する。  
## CD（継続的デリバリー）
CIを拡張した概念で、成果物を検証環境へ自動デプロイする手法。  
CIの成果物を自動で検証環境に反映させることですぐに本番リリース可能な状態に保ち、リリースまでの時間を短縮する。  
さらに発展して、本番環境デプロイまでを自動化するフローを、明示的に継続的デプロイメントと呼ぶこともある。  
クラウド・コンテナの普及、インフラのコード化によってデプロイの自動化が可能になった。  

# Concouse
Pivotalによって開発されたpipeline-basedなCI/CDツール。Goで開発されている。  
UIでどのJobどうしが依存しているのか、どこで問題が発生しているのかを視覚的に把握できるのが特徴。 
## 独自の概念
**pipeline**：外部の状態を表すresourcesと、resourcesと相互作用するjobを主要素として構成された、CI/CDプロセスを表現するもの。  
- 自己完結的に設計されており、「pipelineさえあればOK」な状況を目指している？  
- YAMLで書かれ、`pipeline.yml`として設定されるのが通例。  
  
**resources**：ソースコードや依存関係、デプロイ状況等、Concourseの外部状態を抽象化したもので、バージョン確認、pull, pushできるようになっている。  
**resource types**：各resourcesに関するメタ情報で、`resouces.type`で指定される。  
**jobs**：実際にパイプライン上で何をするのかを指定する要素。一連のstepで構成され、各jobは冪等かつ疎結合な設計であるべきとされる。  

## CLI
Concourseは`fly`コマンドを介してCLIで操作を行う。  
- `fly -t <target> login -n <team> -c <concourse-url>`で、指定したConcourseサーバとteamを`target`として登録しながら認証できる。  
  - targetごとにflyのバージョンが指定されており、外れている場合は`fly -t <target> sync`でCLIのバージョンを同期する必要がある。  
  - fly login詳細：https://concourse-ci.org/fly.html#fly-login  
- `fly -t <target> pipelines`でログイン中のteam配下のpipelineを一覧。  
- `fly -t <target> get-pipeline(gp) -p <pipeline>`で特定のpipelineの詳細を閲覧。  
- `fly -t <target> set-pipeline(sp) -c <local_yaml_file> -p <pipeline>`でpipelineの作成/更新。  
- `fly -t <target> pause-pipeline/unpause-pipeline -p <pipeline>`でpipelineの停止/開始。  
- `fly -t <target> destroy-pipeline(dp) -p <pipeline>`でpipelineの削除。  

## チュートリアル要約
### Hello World
- Concourseにおいて構成可能な最小単位はTaskであり、Pipelineの中では、Jobを構成する連続したStepsの1つ（task step）として実行される  
- `fly -t <target> execute/e --config/-c <config_file>`でconfig_fileに記述されたTaskを実行できる  
- 以下の3つの設定は必須  
  - `platform`: そのtaskの実行環境を表し、linux/darwin/windowsのいずれかが使われる  
  - `image_resource`: `platform`上で動作するコンテナイメージを指定する  
  - `run`: コンテナ内で実際にTaskとして実行するコマンドを`path`で指定する  
- 以下は任意の設定  
  - `params`: key-valueでそのTaskから利用できる環境変数を設定する  
  - `privileged`: これをtrueに設定すると、（platform: linuxの場合は）Taskをroot権限で実行する  
  - `caches`: そのTask実行時にキャッシュされるディレクトリパスを指定する  
- 各Taskは冪等になるよう構成するのが理想  
- 公式：https://concourse-ci.org/tasks.html#tasks

### Taskへの入力について
- `inputs`フィールドを指定することで、指定した名前で渡されたファイル/フォルダをTaskのコンテナ内で利用出来る  
  - `path`を指定することで、そのコンテナ内のどこに`input`を配置するかを指定できる  
- **`task.inputs`が指定されているにも関わらず入力が得られない場合、Taskは失敗する**  
- 実際にTaskのコンテナに`inputs`を渡すには以下の手段がある  
  - そのJob内で実行されたget stepで読み込んだResourceを指定する  
  - 直前のTaskの`outputs`フィールドを利用して渡す  
  - `fly -t <target> e -c <config> -i <input_name>=<path> `でシェルから渡す  
    - `task.inputs`で指定された名前と現在のディレクトリ名が一致している場合、`-i`オプションは不要  
- 渡されたファイルは、コンテナ内で`./<input_name>`というパスで利用できる  
  - `inputs.path`を指定した場合はそのディレクトリを参照する  

### Taskの実行ファイルの指定
- `task.inputs`には、そのTaskが使用するための依存関係だけでなく、Taskの処理そのものを担うスクリプトを渡すことも出来る  
  - Taskの実用上も、コマンドを直接呼び出すケースより、処理を担うシェルスクリプトを`inputs`に渡して実行させるケースのほうが多い  
- `run.path`で、inputsで渡した実行したいスクリプトのパスを指定する  

### ベーシックなパイプライン
- **ほとんどのケースでは、Taskは`fly e`ではなくPipelineにおけるJobの構成要素の1つとして実行される**  
- Pipelineを構成するYAMLファイルでは、`jobs.job.plan`の配下に実行すべきStepが列挙され、Taskもその1つになる  
  - task以外の利用可能なStep一覧：https://concourse-ci.org/jobs.html#steps  
- Pipelineの最小単位は実行すべきJob  
- `fly set-pipeline(sp) <pipeline.yml> -p <pipeline_name>`の形式で Pipelineを作成  
  - デフォルトはPause状態なので、UI/`fly up -p <pipeline_name>`でpauseを解除する  
  - Pause解除後は、UIで+ボタンを押すか、`fly trigger-job(tj) -j <pipeline_name>/<job_name>`でJobが実行される  
  
### Resourceについて
- Concourseにデータの保存/取得をするサービスは無いので、**全ての入出力はConcourse外から提供する必要があり、これをResourceと呼ぶ**  
- PipelineのYAMLファイル中では、個別のResourceは`resources.resource`として定義される  
- どのようなResourceが利用できるのかを定義しているのがConcourseのResource Types  
  - 公式の一覧：https://resource-types.concourse-ci.org/  
  - 最も一般的なResource Typeはgit  
- Resourceを実際にJobの中で利用するためには、get stepでresource.nameを指定して読み込む必要がある  
  - 読み込まれたResourceは、Job内の以後の任意のTaskで、入力として利用できる  
  
### Jobの出力の表示
- `fly watch -j <pipeline_name>/<job_name> --build <build #>`で特定のbuild numberのJobの出力を表示できる  
- `fly builds`で、全てのパイプラインを対象に、最近のビルド結果を一覧表示できる  

### Jobを起動する4つの手段
1. JobのWeb UI右上の+ボタンをクリック（前述）  
2. Resourceの検出からJobを起動（次のセクションで説明）  
3. `fly trigger-job(tj) -j <pipeline_name>/<job_name>`コマンドを実行（`-w`オプションで実行ログをターミナル表示）  
4. Concourse APIへPOSTリクエストを送信  

### Resourceを利用してJobを起動する
- 実務上、Jobが起動される主な方法は、前掲2のResourceの変更の検出がほとんど（ex. masterブランチへのコミットに応じてJobを起動）  
- Resourceの変更を機にJobを起動するには、そのResourceについてのget stepで、`trigger`フィールドを`true`に設定する  
  - `trigger: true`に設定されていると、そのResourceの新しいversion（ex. gitリポジトリへのコミット）が利用可能になった際に自動でJobが起動する  
- Jobを定期実行させることを主目的とした、timeというResource Typeが組み込みで存在する  
  - `resource.source.interval`で設定された間隔で、新しいversionが発行されるというResource  
  - `trigger: true`にすることで`interval`で指定した間隔でJobを定期実行できるという仕組み  
- Web UI上の表現として、`trigger: true`がセットされているResourceは、実線でJobに接続される  
  - triggerではないResource（デフォルト）は、破線でJobと繋がれている  

### Taskの出力を後続Taskの入力として使う
- `task.outputs`を指定することで、そのTaskの出力を後続の`task.inputs`から利用できるようになる  
- `task.outputs`を設定すると、Task実行前に`task.outputs.name`の名前でディレクトリが作成される  
  
### Taskの成果物を外部へアップロードする
- `task.outputs`を利用することで、後続のTaskのみならず、Concourse外に成果物を渡すことも可能  
- get stepがResourceをfetchするのと対照的に、put stepはResourceの更新が可能であり、その際に`task.outputs`を利用出来る  
  - つまり、taskの成果物を使ってConcourse外部のResourceの内容を変更できる  
  - ex. git Resource Aについて、(get: A)→(task.inputs: A)→(process)→(task.outputs: B)→(put: B to A)のように、一連のstepを通じて、リポジトリを変更＆コミットしてリモートへpushできる  
  
### パラメータを利用する
- credentialなどの情報を外部から与えられるパラメータとして記述することで、`pipeline.yml`で秘匿情報を平文記述するのを避けることが出来る  
  - ex. `params: {some_key: ((parameter))}`のように、パラメータ化したい値を(())で囲えばOK  
- 値をパラメータ化する際、デフォルト値は設定できない  
- `fly sp -c pipeline.yml -p <pipeline_name> -v <key>=<value>`のように`-v`オプションを使うことで、Pipeline設定時に変数として与えることが可能  
  - `-l`オプションでローカルのYAMLファイルを使って渡すことも出来る  
- ナイーブなパラメータは以下の問題があり、credential managerで解決可能  
  - パラメータを変更する際、共通のパラメータを持つPipelineに対して逐一fly spを再実行する必要がある  
  - そのパイプラインが存在するTeamに属する人は、実際に動作しているpipeline.ymlを`fly get-pipeline -p <name>`でダウンロード出来るので、必ずしもセキュアではない  
  
### 複数のJob間でResourceを共有する
- 実用的なPipelineは、各Job間で実行結果を共有できるように作られる必要がある  
  - ex. 正常終了したJobの結果（Job実行後のResource）を元に、他のJobを実行する  
- 以下の設定を行うことで「Job1が変更を加えたResourceに基づいて、自動的にJob2を実行する」ようなPipelineが作れる  
  - `Job1.task.outputs`、`Job1.put`を使ってResourceを更新する  
  - そのResourceについて、`Job2.get`で`trigger: true`を設定し、変更を検知できるようにする  
  - triggerの設定に加えて、`Job2.get`で`passed: Job1.name`を設定することで、`Job1.task`によって更新されたResourceを利用するように明示する  
- `job.get.passed`：指定したJobを介して更新されたResourceのバージョンだけが、trigger, fetchの際に考慮される  
  - ex. あるgit Resourceについて`Job2.get.passed: [Job1]`のように設定すると、Job2はJob1によって行われたコミットだけを利用して実行され、たとえもし新しく手作業でのコミットが存在していても、それは無視される  
  - これによってJob間の依存関係を安全に定義できる  
  
### Credential Managerを使う
- Concourseは以下のCredential Managerに対応しており、Pipeline中のパラメータをそれらから動的に取得することが出来る  
  - Cloud Foundry CredHub  
  - Hashicorp Vault  
  - Amazon SSM/Amazon Secrets  
- パラメータ取得の際には、それらのCredential Managerの次のパスを順番に調べるので、汎用的な情報は同じTeam配下のPipelineで共有することが出来る
  - `/concourse/<team_name>/<pipeline_name>/<parameter>`  
  - `/concourse/<team_name>/<parameter>`  
- パラメータがManager側で更新されると、Concourseは次回Job実行時にそれを自動的に反映する  
- どのCredential Managerを利用する場合でも、Concourseからは同じようにやり取りできる  

### Dockerイメージのビルド・利用
- 公式ではdocker-image Resourceが用意されている  
  - Dockerイメージを**ビルドするパイプラインでは、docker-image Resourceではなくregistry-image Resourceで置換するべき**とされている  
- registry-imageは、Dockerイメージの検証、fetch、pushしか行わない（buildしない）  
  - 純粋なGoで書かれており、Docker CLIやDocker daemonを含まない  
  - ビルドは、tarファイルを生成するoci-buildのようなTaskで行われる  
  - registry-image：https://github.com/concourse/registry-image-resource  
#### oci-build-taskについて
- Open Container Initiative(OCI)準拠なImageをビルドするためのConcourse Taskを表すDocker Image  
  - OCI：コンテナ技術の実装の業界標準策定を目的に提出された発議で、ランタイムとイメージの2つについて標準仕様を策定している  
- 「Resource同様に再利用可能なTaskを実装しよう」という提案のPoCのために作られたもので、仕様が固まれば必要な設定も変わりうるらしい
  - 例えば、現状は`privileged: true`に設定しないとビルドは失敗する  
- oci-build-taskの`config`の書き方  
  - `image_resource`: `type: registry-image`、`source.repository: vito/oci-build-task`を指定する  
  - `params`: build実行のコンテナで利用できる任意の環境変数で、主に以下が使われる  
    - `CONTEXT`: デフォは`.`で、ビルドコンテキストに使われるディレクトリのパスを表す  
    - `DOCKERFILE`: Dockerfileのパス  
    - `BUILD_ARG_<VAR>`: 接頭辞BUILD_ARG_を持つ環境変数はbuild argに使われる  
    - `UNPACK_ROOTFS`: これをtrueにすると、ビルドしたImageのファイルシステムが実際に展開される（後続Task stepでそのまま利用できる）  
  - `inputs`: Taskを実行するために必要な入力なので、基本的に`CONTEXT`の値と同じになる  
  - `outputs`: 基本的に`name: image`に設定される  
    - 実際に生成されるimageディレクトリの中には以下が含まれる  
      - `image.tar`: OCI imageのtarball（圧縮済みtarファイル）で、registry-image Resourceを通して外部Registryにアップロード可能  
      - `digest`: OCI Image configurationのダイジェスト  
    - paramsで`UNPACK_ROOTFS`を設定していると以下のファイル/ディレクトリも作成される  
      - `rootfs/*`: そのImageのファイルシステムが展開されたもの  
      - `metadata.json`: そのImageの環境/ユーザ設定を記述したJSONファイル  
  - `caches`: `path: cache`に設定することでビルドTaskにキャッシュを利用できる  
  - `run`: `path: build`と設定することで、このTaskからImageのビルドを実行できる  
- 公式：https://github.com/vito/oci-build-task  


