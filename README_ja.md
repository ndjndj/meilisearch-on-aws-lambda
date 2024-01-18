# Meilisearch on Amanzon Lambda

Meilisearch のサーバーレス実行環境を AWS 上に構築するためのリポジトリです。  

## Abstract 

- Meilisearch の実行環境を Lambda Web Adapter を用いて AWS Lambda で動かせるようになりました。
- インデックスの保管場所に Amazon EFS を利用しました。
- Amazon EFS の読み取り量がえらいことになりました。

→ 実用はむずかしそうという結果が得られた。

## Introduction 

Meilisearch とは、Meili 社および多くの Contributor によって開発されているオープンソースの全文検索エンジンです。  
ミリ秒単位の非常に高速な検索パフォーマンスが特徴で、多くの支持を得ています。  
また、コミュニティが活発なところも私の個人的に好きなポイントの一つです。

このリポジトリでは、Meilisearch のサーバーレス実行環境を構築する方法を紹介したかったんですが、なんとも本番環境ではいまいち使えなさそうなことがわかってきたので供養します。  

## Notice

以下の AWS サービスを利用することにより、$1 未満～ の利用料金が発生することに注意してください。  
作成する Meilisearch のインデックスサイズ、インデックス作成の頻度、検索が実行された回数により、AWS に対する利用料金は変わります。

細かい利用料金に関しては、該当のページもしくは [AWS Pricing Calculator](https://calculator.aws/#/) をご覧ください。

- [Amazon EFS](https://aws.amazon.com/jp/efs/pricing/)
    - Meilisearch インデックスの保管場所として。
- [Amazon EC2](https://aws.amazon.com/jp/ec2/pricing/) (option)
    - Meilisearch インデックスを Amazon EFS に転送するため。
- [AWS DataSync](https://aws.amazon.com/jp/datasync/pricing/) (option)
    - Meilisearch インデックスを Amazon EFS に転送するため。

## Why use serverless?

全文検索エンジンを導入したいと考えたとき、どのような手段を考えるでしょうか。  
一つは、Meilisearch Cloud, Algolia のようなクラウドサービスを導入することで、  
もう一つは、全文検索エンジンをセルフホスティングすることでしょうか。  

どちらにしても、いくらかのランニングコストを支払い続ける必要がある一方で、  
全ての製品に十分な予算が割り当てられているとは限りません。  
特に、個人で開発・運営されているような製品や小規模な製品において、それは顕著だと思います。  
開発者たちは、よりよい製品を作りたいという目標を掲げると同時に、継続した開発・運営のために低コストな方法を模索しています。  

今回は、できるだけ低コストで全文検索エンジンを導入したいというニーズを叶えられるかもしれないと考えた人の挑戦をつづります。

## Usage

サーバーレス環境作成のための手順をいくつかのステップに分けると以下のようになります。
また、検索対象となるデータはローカル(非 Lambda 環境)で作成する必要があります。

1. Meilisearch 実行環境の [Dockerfile](./prod/Dockerfile) を作成します。  
2. 1 で作成した [Dockerfile](./prod/Dockerfile) に Lambda Web Adapter を適用し、build image します。
    - 環境設定の類を設定する
3. 2 で作成した Dockerimage を AWS Lambda にデプロイします。
4. AWS Lambda に Amazon EFS をマウントします。
5. 4 でマウントした EFS のアクセスポイントに data.ms をインポートします。
    - インポート方法はなんでも構いません

## Not suitable case 

以下のようなケースでは、今回の実装は向いていないと考えられます。  
理由は後述します。

- リアルタイムで Meilisearch インデックスとメイン DB を同期させたい場合
    - DynamoDB ストリームを利用したインデックス作成
    - etc.

- アクセスの多いサービスでの実装

## Suitable case 

逆に以下のようなケースでは、今回の実装で効果があるかもしれません。  
理由は後述します。

- 更新頻度が低いデータを Meilisearch インデックスとする場合
    - 個人運営しているブログ
    - スポーツ選手名鑑
    - 食品のカロリー表
    - コーポレートサイト
    - etc.

- SSG で構築されたサイト

## Suitable or Not suitable 

議論するキーポイントの一つとして永続ストレージとして利用している Amazon EFS の存在があります。  

前提として、Meilisearch では、data.ms 直下のファイルの更新を都度行っているらしく[^1]、 Lambda で使おうとすると、Read-only file system error を防ぐために、data.ms/ を tmp 直下以外に置くことはできません。 
しかし、Lambda では、tmp 直下のファイルはある程度時間がたつと消えてしまうので、うまく検索を実行することができませんでした[^2]。  

上記のような背景から、今回は Amazon EFS の利用を決めました。
しかし、EFS に data.ms をマウントするため、読み込み時の容量が非常に大きくなりがちです。

Lambda は、一定のタイミングでインスタンスを破棄するので、何度も data.ms を読み込むことになります。

[^1]: 更新のタイミングはキャッチアップできていません。ファイルの更新日ベースの判断です。

[^2]: Index not found. が発生する。もしかしたら検証の余地は残っているかもしれません。そもそもデータが image に残っていないのかも

## 最後に 

Meilisearch の高い検索性能がなければサーバーレスで構築しようなどとは考えもしませんでした。Meilisearch に関わる全ての Contributor に感謝します。

今回、低コストで全文検索エンジンを導入する選択肢を増やすために挑戦してみましが、なんとも言えずの結果になりました。
しかし、やってみてダメだと分かったこととはじめからダメだと言われたことは違います。  
きっと改善の余地はあるし、  
Meilisearch の性能を実感することができたし、Lambda Web Adapter の便利さも知れたので僕の血肉になっていることでしょう。


意見や改善点を歓迎します。  
[Issue](https://github.com/ndjndj/meilisearch-on-aws-lambda) やコメントください。