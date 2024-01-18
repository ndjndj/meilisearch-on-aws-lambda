# Meilisearch on Amanzon Lambda

Meilisearch のサーバーレス実行環境を AWS 上に構築するためのリポジトリです。  

## Introduction 

Meilisearch とは、Meili 社および多くの Contributor によって開発されているオープンソースの全文検索エンジンです。  
ミリ秒単位の非常に高速な検索パフォーマンスが特徴で、多くの支持を得ています。  
また、コミュニティが活発なところも私の個人的に好きなポイントの一つです。

このリポジトリでは、検索用途に特化した Meilisearch のサーバーレス実行環境を構築する方法を紹介します。  

今回のショーケースにおいて、Meilisearch における銀の弾丸であると主張するつもりはもちろん毛頭ないが、特定のユースケースにおいては価値を発揮すると考えている。

## Notice

以下の AWS サービスを利用することにより、$1 未満～ の利用料金が発生することに注意してください。  
作成する Meilisearch のインデックスサイズ、インデックス作成の頻度により、AWS に対する利用料金は変わります。

細かい利用料金に関しては、該当のページもしくは Amazon Calc をご覧ください。

- [Amazon EFS](https://aws.amazon.com/jp/efs/pricing/)
    - Meilisearch インデックスの保管場所として。
- [Amazon EC2](https://aws.amazon.com/jp/ec2/pricing/) (option)
    - Meilisearch インデックスを Amazon EFS に転送するため。
- [AWS DataSync](https://aws.amazon.com/jp/datasync/pricing/) (option)
    - Meilisearch インデックスを Amazon EFS に転送するため。

## Cannot in this case 

- リアルタイムで Meilisearch インデックスとメイン DB を同期させたい場合
    - ex. DynamoDB ストリームを利用したインデックス作成

## Suitable case 

- 更新頻度が低いデータを Meilisearch インデックスとする場合
    - ex. 

## Why use serverless?

全文検索エンジンを導入したいと考えたとき、  
最初に思い浮かぶのは、Meilisearch Cloud, Algolia のようなクラウドサービスを導入することか、もしくは、セルフホスティング可能な全文検索エンジンを自分が管理するサーバー上にホストすることでしょう。  
どちらにしても、いくらかのランニングコストを支払う必要があります。

しかし、全ての製品に十分な予算が割り当てられているとは限りません。  
皆がよりよい製品を作りたいと考えると同時に、継続して開発を続けるためにより低コストでそれを行う方法を模索しています。  

特に、個人で開発・運営されているような製品や小規模な製品において、

今回のショーケースでは、できるだけ低コストで全文検索エンジンを導入したいというニーズを叶えることができる一つの選択を提示します。

中小規模な Web サイトや個人で管理されるような Web サイトにおいても、  
全文検索エンジンの導入ニーズは高いだろうと考えました。  



そのために、
AWS Lambda は、多くの無料枠を与えてくれています。  さらに、EFS を
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
