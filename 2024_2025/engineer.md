


# 採算がとれる動き

## 「docker run」に対応する記事の作成 (️⭐⭐︎︎⭐︎⭐︎⭐︎)

現状、ブログの「docker run オプション」ではシェアを取れている。


| 上位のクエリ |	クリック数 |	表示回数 | 掲載順位
| - | - | - | - |
| docker run オプション	| 876 |	3,053 |	2.5 | 
| docker run | 472 | 10,316 | 4.8 |

しかし、「docker run」キーワードによる検索結果は、表示回数の割にはクリック数が低い。

これは、「docker run オプション」専用には




# rei

reiはポルトガル語で王様という意味らしい。











# AWS でdockerを動かす方法

## AWSでdockerコンテナを動かすために「最低限」必要なサービス

<img src="https://github.com/minegishirei/store/blob/main/docker/aws/ecs/ecs-lifecycle.png?raw=true">

AWSでDockerコンテナを動かすために「最低限」必要なサービスは以下の通りです。

- `Amazon Elastic Container Service (ECS)` または `Amazon Elastic Kubernetes Service (EKS)`
    - ECS,EKSはDockerコンテナを実行するサービスの本体です。
    - ECSは比較的シンプルなコンテナ管理サービスです。
    - EKSはKubernetesを運用しているため、大規模なシステムでもオーケストレーションが可能になっているサービスです。

使い分けをまとめると、「EKSを使うときは大規模なサービス」「ECSは小規模なサービス」として覚えておくとよいと思います。

また、ECS Fargateを使用する際には ECS Taskdefineを作成する必要があります。

- `ECS TaskDefine`
    - タスク定義は1つ以上のコンテナ定義を含みます。各コンテナ定義には、Dockerイメージの指定、ネットワークポートのマッピング、ボリュームのマウントなどの設定が含まれます。
    - コンテナが使用するCPUおよびメモリリソースの割り当てを指定します。
    - エントリーポイント、または実行コマンドの上書をします。
    - そのほか、ネットワーク設定やデータボリュームの指定を行います。

ローカル開発環境では`docker run`コマンドのオプションを指定したり`docker-compose.yml`ファイルに実行時のオプションを指定します。
AWSでコンテナを動かすためには`ECS taskdefine`でこれらのオプションを定義する必要があるのです。

- `Amazon Elastic Container Registry (ECR)`
    - ECRはプライベートなコンテナレジストリです。ビルドしたDockerイメージを保存しておくのに使います。
    - Docker Hubにイメージを保存するのもよいですが、より機密性が求められる業務システムはプライベートなレジストリにDockerイメージを保存するケースが多いです。

業務システムとして使用する場合はECRは必須となります。

Dockerコンテナを動かすために最低限必要なサービスは`ECS service` `ECS Taskdefine` `ECR`のみです。
これらをまとめると、以下のような図面になります。

<img src="https://github.com/minegishirei/store/blob/main/docker/aws/ecs/ecs-lifecycle.png?raw=true">





## AWSでdockerコンテナを動かす際に「一般的に必要とされる」サービス

最低限必要なサービスは`ECS`と`ECR`のみです。

ですが上記の二つのサービスだと以下の負担が発生します。

- ソースコードのBuildを手作業で行う必要がある。
- Blue/Green Deployができない（システムの入れ替え時にダウンタイムが発生する）
- ソースコードの変更の検知からサービスの入れ替えをしてくれない。

これらの課題を解決するためのサービスが AWS Codeサービス群です。

- `AWS CodePipeline` 
    - ソースコードのビルド、テスト、デプロイメントを自動化するためのサービスです。
    - CodecommitやGithubなどの特定のリポジトリの特定のブランチの変更を検知し、Codebuildやcodedeployに命令を出します。
    - 手動でのビルド作業が不要になり完全自動化されたCICD環境を整えることができるのです。
- `AWS CodeDeploy`
    - Blue/Green Deployを可能にするデプロイメントサービスです。
    - 新規デプロイの際には、新しいコンテナを立ち上げコンテナのヘルスチェック確認を行った後で、ロードバランサーの接続先を切り替えます。
    - 新しいバージョンのアプリケーションをデプロイする際にダウンタイムを最小限に抑えることができます。
- `AWS Codebuild`
    - CodeBuildを使用すると、コンテナイメージのビルドプロセスを自動化できます。
    - ソースコードの変更があったときに自動的にビルドがトリガーされ、最新のコンテナイメージが作成されます。

これらをまとめると、以下のような図面になります。

<img src="https://github.com/minegishirei/store/blob/main/docker/aws/ecs/fargate_pipeline.png?raw=true">
















## AWS codebuild

https://aws.amazon.com/jp/codebuild/














