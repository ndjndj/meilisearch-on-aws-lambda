# meilisearch-on-aws-lambda

Meilisearch のサーバーレス実行環境を AWS 上に構築するためのリポジトリです。  

## Introduce 

Meilisearch とは、Meili 社および多くの Contributor によって開発されているオープンソースの全文検索エンジンです。  
ミリ秒単位の非常に高速な検索パフォーマンスが特徴で、多くの支持を得ています。

このリポジトリでは、検索用途に特化した Meilisearch のサーバーレス実行環境を構築する方法を紹介します。

## Why use serverless?

全文検索エンジンを導入したいと考えたとき、  
最初に思い浮かぶのは、いくらかのランニングコストを支払って Meilisearch Cloud, Algolia のようなクラウドサービスを導入するか、もしくは、セルフホスティング可能な全文検索エンジンを自身が管理するサーバー上にホストすることでしょう。

しかし、全ての製品に十分な予算が割り当てられているとは限らず、皆がよりよい製品を作りたいと考えると同時に、より低コストでそれを行う方法を考えています。

今回のショーケースでは、できるだけ低コストで全文検索エンジンを行いたいというニーズを叶えることができる一つの選択を提示します。

中小規模な Web サイトや個人で管理されるような Web サイトにおいても、  
全文検索エンジンの導入ニーズは高いだろうと考えました。  



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

    SSG を利用しているのなら

## Usage 

大きく分けると

Meilisearch の実行環境の DockerImage を作成します。  
この DockerImage には Lambda Web Adapter を適用されており、  
これにより AWS Lambda で Meilisearch が起動可能となります。  
DockerImage を AWS Lambda に適用します。  
AWS Lambda には Meilisearch のデータがインポートされた AWS EFS がマウントされています。

検索対象となるデータはローカル(非 Lambda 環境)で作成する必要があります。

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
