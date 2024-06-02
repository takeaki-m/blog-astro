---
title: "AWS Kinesis Data Stream Memo"
description: 'post abount aws kinesis'
pubDate: '2023-07-17T14:45:11+09:00'
---

### キューとの違い_KinesisとSQS
- データの削除
  - キュー
    - キュからデータを取り出した際に、処理が完了すれば取り出したデータは削除してキューの先頭は別のデータになる
  - Kinesis
    - 取り出したデータも保持する
    - デフォルト
      - 24時間
    - 最大
      - 365日(8760時間)
      - 24時間を超えた保持期間を設定した場合には追加料金が発生
    - [Title](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/kinesis-extended-retention.html)
[[AWS]kinesisまとめ - Qiita](https://qiita.com/yShig/items/500d4139efed91bc432d)



基本的に以下の資料を見れば全体の概要はわかる
いつの間にやらBlackBeltでもわかりやすい資料が出ていた!(2023/04)
[Amazon Kinesis Data Stream -- AWS Black Belt Online Seminar](https://pages.awscloud.com/rs/112-TZM-766/images/AWS-Black-Belt_2023_AmazonKinesisDataStreams_0430_v1.pdf)

特にデータ取得の設定に関して実例も踏まえてわかりやすい記事
[AWS kinesisまとめ - ブログ - 株式会社Smallit（スモーリット）](https://smallit.co.jp/blog/a1410/)
[[AWS]kinesisまとめ - Qiita](https://qiita.com/yShig/items/500d4139efed91bc432d)


### Consumerの設定
- イテレータの取得方法(=取得するデータの選択)です。 今回はTRIM_HORIZONを指定しているので最も古いデータ、つまり最初に送信したデータを取得できるハズです。

| shard-iterator-type | データの選択方法 |
|---|---|
| AT_SEQUENCE_NUMBER| あるシーケンス番号|
| AFTER_SEQUENCE_NUMBER| あるシーケンス番号の後|
| TRIM_HORIZON |最も古いレコード|
| LATEST| 最も新しいレコード|
| AT_TIMESTAMP| 指定したタイムスタンプ|

> ベストプラクティスからは少し離れてしまいますが、コンシューマがイテレータを取得する際の設定項目であるデータの取得範囲(shard-iterator-type)がLATESTである場合は、コンシューマアプリケーションを起動してからプロデューサアプリケーションを起動する必要があります。
> 逆に、shard-iterator-typeがTRIM_HORIZONならば先にプロデューサを起動します。


以下自分のための用語のまとめ

### Amazon Kinesis Data Streamの用語
- Kinesis Data Stream
  - shardの集まり
  - shardはdata recordsの集合
- Data Record
  - Kinesis Data Streamに保存されているデータ
  - 構成要素
    - sequence number
    - partition key
    - data blob
      - 不変のデータ
      - 最大1MBまで
- Capacity Mode
  - 処理能力の管理設定と変更設定
  - 二つの種類がある
  - on-demand
  - provisioned
  - on-demand
    - 必要なthroughputに応じて、自動的にshardsが管理
  - provisioned
    - shardsの数を自分で設定、管理
    -  
- Retention Period
  - streamに追加されたdataの保持期間
  - 24 hours - 365days(8760 hours)
  - 24 hoursを超えると追加料金が発生する


- Shard
  - それぞれが独立したdataのシーケンス
  - 一つのStreamは1つ以上のshardで構成される
  - 処理能力(shard一つあたり)
    - 読み取り
      - 1秒あたり、「5つの読み取りリクエスト・処理データの合計2MB」
    - 書き込み
      - 1秒あたり、「1000レコードの書き込み・書き込みデータの合計1MB」
  - streamの処理能力は、streamを構成するshardの合計処理能力と等しい
- Re-sharding
  - shardの分割と結合
  - 分割
    - 1->2
  - 結合
    - 2->1
  - shardが増えるとストリーム内のデータ容量が増える→コスト増加
[ストリームをリシャーディングする - Amazon Kinesis Data Streams](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/kinesis-using-sdk-java-resharding.html)


- sharding
  - hot
    - 想定より過多なデータを受け取っている
  - cold
    - 想定より過小なデータを受け取っている


- hash key
  - hash keyに基づいて、shardに割り当てられる
  - data recordのpartition keyのMD5 hash
    - MD5
      - parittion keyの計算で使われることが多い(セキュリティ脆弱性により、暗号化では使われなくなった)
      - [MD5 - Wikipedia](https://en.wikipedia.org/wiki/MD5)
  - partition keyが同じ場合、hash keyも同じ
- shardを分割した際のhash keyの計算
  - 親shardのhash key のrageを取得
  - hash key rangeを2分割して、新しい子shard二つに設定する
- shardの結合
  - hash keyが隣接している親shard二つを、一つの子shardに結合できる
  - OPNE状態のshardのみが対象


- shardの結合・分割を実施した後の操作方法
  - streamが再度アクティブになるまで待機する
  - ※AWS consoleからも確認できるのか？
  - 親shardにデータが残る可能性があるため完全に処理をすること
    - どのような場合にdataが残るのか
    - 子shardに引き継がれるわけではないのか？
    - ※ここは動作確認が必要
[リシャーディング後 - Amazon Kinesis Data Streams](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/kinesis-using-sdk-java-after-resharding.html)

## Kinesis Producer Library
- 後ほど調べて記載する

## Kinesis Client Library
[Using the Kinesis Client Library](https://github.com/awslabs/amazon-kinesis-client/blob/v1.x/src/main/java/com/amazonaws/services/kinesis/leases/impl/Lease.java)
- Worker
  - KCL Cosumer application instanceがデータを処理するために使用する抽象的なクラス
  - Kinesis Data Streamからdataの取得、処理など主要な処理を担う
- Lease
  - workerとshardの関連を定義するデータ
  - 一つ一つのshardは、leaseKeyによって、workerと紐づけられている
  - 例
    - shardが4つ、ConsumerApplication-AとWorker-Aがある場合
      - Worker-Aが、shard1,2,3,4のleaseを保持する
    - shardが4つ、ConsumerApplication-AとWorker-A & Consumer-BとWorker-Bがある場合
      - Worker-AとWorker-Bは、一つのshardのleaseを同時に保持できない
      - 一つのworkerがleaseを手放すと、別のworkerがleaseを保持する
  - 参考
    - [KCL1](https://github.com/awslabs/amazon-kinesis-client/blob/v1.x/src/main/java/com/amazonaws/services/kinesis/leases/impl/Lease.java)
    - [KCL2](https://github.com/awslabs/amazon-kinesis-client/blob/master/amazon-kinesis-client/src/main/java/software/amazon/kinesis/leases/Lease.java)
- Lease Table
  - KCL consumer applicationのworkerによって処理されているshardを管理するDyanamoDBのテーブル
  - KCL consumer appliationが実行されている間は、最新のkinesis data streamの情報を同期している必要がある
- Record Processor
  - KCL consumer applicationが取得したデータの処理方法
  - KCL consumer appliationが一つのworkerを起動し、そのworkerは全てのshardを処理するprocessorを起動する
