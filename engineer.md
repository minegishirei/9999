


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

AWSでDockerコンテナを動かすために「最低限」必要なサービスは以下の通りです。

- Amazon Elastic Container Service (ECS) または Amazon Elastic Kubernetes Service (EKS)
    - ECS,EKSはDockerコンテナを実行するサービスの本体です。
    - ECSは比較的シンプルなコンテナ管理サービスです。
    - EKSはKubernetesを運用しているため、大規模なシステムでもオーケストレーションが可能になっているサービスです。

使い分けをまとめると、「EKSを使うときは大規模なサービス」「ECSは小規模なサービス」として覚えておくとよいと思います。

- Amazon Elastic Container Registry (ECR)
    - ECRはプライベートなコンテナレジストリです。ビルドしたDockerイメージを保存しておくのに使います。
    - Docker Hubにイメージを保存するのもよいですが、より機密性が求められる業務システムはプライベートなレジストリにDockerイメージを保存するケースが多いです。

業務システムとして使用する場合はECRは必須となります。

Dockerコンテナを動かすために最低限必要なサービスは`ECS`と`ECR`のみです。


## AWSでdockerコンテナを動かす際に「一般的に必要とされる」サービス

最低限必要なサービスは`ECS`と`ECR`のみです。

ですが上記の二つのサービスだと以下の負担が発生します。

- ソースコードのBuildを手作業で行う必要がある。
- Blue/Green Deployができない（システムの入れ替え時にダウンタイムが発生する）
- ソースコードの変更の検知からサービスの入れ替えをしてくれない。

これらの
















## AWS codebuild

https://aws.amazon.com/jp/codebuild/














