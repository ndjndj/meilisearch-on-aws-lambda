# meilisearch-on-aws-lambda

Meilisearch のサーバーレス実行環境を AWS 上に構築するためのリポジトリです。  

## Introduce 

Meilisearch とは、多くの Contributor によって開発されているオープンソースの全文検索エンジンです。  
高速な検索性能と　が特徴で、多くの支持を得ています。

検索用途に特化した Meilisearch のサーバーレス実行環境を構築する方法を紹介します。

## Why serverless?

中小規模な Web サイト、個人で管理されるような Web サイトのうち、  
更新頻度が低いがデータ量が多いような Web サイトにおける検索体験向上のために、全文検索エンジンの導入のニーズが高いだろうと考えました。  

全文検索エンジンを導入するには、Meilisearch Cloud, Algolia のようなクラウドサービスを導入する、もしくは、全文検索エンジン自体を自身が管理するサーバー上にセルフホスティングを行うことになるでしょう。
どちらにしても、いくらかのコストはかかります。
しかし、そのようなサイトに多くの予算が割り当てられている場合は多くありません。

そのために、
AWS Lambda は、多くの無料枠を与えてくれています。さらに、EFS を
そこで、

## Usecase

更新頻度が低いようなサイトに使えるでしょう。
逆に、リアルタイムにインデックスを行うようなケースには向いていないでしょう。  
なぜなら、


- 例
    - 個人ブログサイト
    - 

    SSG を利用しているのなら、

## Usage 

大きく分けると

### Docker 

### EFS

### Lambda 

1. Dockerfile の構築
2. DockerImage の作成、AWS ECR へのプッシュ
3. AWS Lambda に 2. で作成した Image を適用
4. AWS Lambda に EFS ディレクトリをマウント


## 

意見や改善点を歓迎します。  
Issue か Message(X or Gmail) ください。

## Acknowledgement 

Meilisearch の高い検索性能と省　がなければサーバーレスで構築しようなどとは考えもしませんでした。Meilisearch に関わる全ての Contributor に感謝します。

