# 作業記録メモ

## やりたいこと

* koelをaws上で動かしたい
  * koelはecsで
  * dbはrdsで
    * https://ec2spotworkshops.com/ja/running-amazon-ec2-workloads-at-scale/seed_db.html を参考に
* 最終的にはterraformとかで再現可能に

## dbを立てる

なにができていたらいい？

* 起動できていること
* DB直アクセスはできない
* 作成したユーザーとパスワードで入れること

どうやったらできそうか？

チュートリアル見ればできそうでは？

Amazon RDS チュートリアルおよびサンプルコード https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/CHAP_Tutorials.html より

```
チュートリアル: DB インスタンス用の Amazon VPC を作成する

同じ VPC 内の Amazon EC2 インスタンスで実行されているウェブサーバーとデータを共有する Amazon Virtual Private Cloud (VPC) に DB インスタンスを含める方法を学びます。

チュートリアル: ウェブサーバーと Amazon RDS DB インスタンスを作成する

PHP を使用する Apache ウェブサーバーのインストールと、MySQL データベースの作成を説明します。ウェブサーバーは Amazon Linux を使用して Amazon EC2 インスタンスで実行され、MySQL データベースは MySQL DB インスタンスと です。Amazon EC2 インスタンスと DB インスタンスの両方が Amazon VPC で実行されます。
```

これでいけそう。
やってみよう。

## チュートリアルに沿ってrdsをたててみる

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/CHAP_Tutorials.WebServerDB.CreateVPC.html
をみながら

> DB インスタンスはウェブサーバーでのみ必要で、パブリックインターネットには必要ない

そうそう。
これを実現したい。

例では米国西部 (オレゴン) リージョンを使用しているので、そのままで。

us-west-2な

タグだけあとからたどりやすいように project koelとタグだけつけとく。

35.85.78.32

気になったので、料金を調べた。
https://aws.amazon.com/jp/premiumsupport/knowledge-center/elastic-ip-charges/
elastic ipは使われていないと請求されるよう。
つかわないときはけしておかないと。


VPC

名前タグの自動生成はkoelとしておいた。
これを設定しておくとVPC内の関連リソースのNameタグがkoelに

vpc名は上記名前タグの自動生成といれたら勝手に
koel-vpcとなった

subnetのip cidr割当は、「Customize subnets CIDR blocks」をクリックすると設定欄がでてきた。
デフォルトアベイラビリティーゾーンが2つになっているが、
チュートリアルにそって、1とする。
そんなに高い可用性もいらないし。
デフォルト値があったが、チュートリアルにそってCIDRを設定


DNS ホスト名を有効化はyes
DNS 解決を有効化　はチュートリアルでは触れられていなかったが、デフォルトでチェックが入っていたためそのままに。
Elastic IPの割当は、ウィザードのバージョンが違うからか、項目がなかったので、何もせず。

作成ってしたら、サブネットなども含めてVPCができたよう。

DB用に追加のサブネットが必要だと

作ったサブネットは、azは先ほどつくったものとは別だが、
同じroute tableを設定した。

> パブリックウェブサーバーの VPC セキュリティグループを作成する

とあるのだが、先ほどVPCでパブリックサブネットを作ったときに、セキュリティグループができている。
設定が問題ないかだけチュートリアルに沿って確認する。

危ないなインバウンドルール全部許可になっていたぞ。

> プライベート DB インスタンスの VPC セキュリティグループを作成する

特定のSecurityGroupからのアクセスを許可できるのか。
知らなかった。
今回はWebサーバー用のSecurityGroupから許可。

> DB サブネットグループを作成する
次はこれ

RDSでDBサブネットグループを作成する

VPCは先ほど作ったものを紐づける。

サブネットを追加する。
us-west-2a 10.0.1.0/24
us-west-2c 10.0.2.0/24

チュートリアルではus-west-2bに10.0.1.0/24となっているけど、
ウィザードではこのようにしかできなかったし、aにひもづくのは10.0.1.0/24としていたので、記事のほうが誤りか？

## ウェブサーバーと Amazon RDS DB インスタンスを作成する

DBインスタンスを作成。
可用性はそこまで求めていないため、開発・テスト用で単一のDBインスタンスにした。

DBインスタンス識別子はkoel-db-instance

db.t3.microで作成

ストレージはデフォルトで、自動スケーリングなしで。
VPCは今まで紐づけていたものを。

セキュリティグループはkoel-db-sg

データベースはkoel
USER koel
passwordは適当に

koel-db-instance.cabnsxqlwimz.us-west-2.rds.amazonaws.com

EC2インスタンスたてて、mysqlクライントyumで入れて
rdsに繋げられた。

