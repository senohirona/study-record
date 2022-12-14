# what
- kafkaについて基本的な概要や調べたことについて自分なりにまとめたもの
- 調べること
    - 基本概要
    - データが削除される際の仕組みについて



# kafka(Apache Kafka)とは
- 複数台のサーバーで大量のデータを処理する分散メッセージングシステム(OSS)
    - 送られてくるメッセージ(データ)を受け取り、受け取ったメッセージを別のシステムやデバイスに送るために使われる
- 特徴(メリット)
    - 取り扱うデータ量に応じてシステムをスケールさせることができる
    - 受け取ったデータを「ディスクに永続化」できるため、任意のタイミングでデータを読み出せる
    - プロダクト間あるいはシステム間をつなぐハブとして機能する
    - 「メッセージの送達保証」ができるため、データロストを心配しなくて済む

## 基本の構成要素
![kafka概要図](./kafka%E6%A6%82%E8%A6%81%E5%9B%B3%20(1).png)


kafkaの基本要素は5つ
- Broker
    - データを受信・配信するサービス
- Message
    - kafka内で扱うデータの最小単位。Kafkaが中継するログの1行1行やセンサーデータなどがこれに当たる。Messageにはkeyとvalueをもたせられ、Message送信時のパーティショニングで利用される
- Producer
    - データの送信元。Brokerへメッセージを送信する
- Consumer
    - Brokerからメッセージを取得するアプリケーション
- Topic
    - メッセージを種別(トピック)ごとに管理するためのストレージ。Broker上に配置され、管理される。
    - ProducerやConsumerは特定のTopicを指定してメッセージの送受信を行うことで、単一のkafkaクラスタで多種類のメッセージの中継を実現している


## データ削除の仕組み
前提として、kafkaは受信したメッセージをディスクに永続化し、Consumerはbrokerで保持されていれば過去のデータを遡って読み出すことが可能。
しかし、現実的問題としてストレージ容量には限りがあるため、データを無期限無制限で保持できるわけではない。
データの削除には以下の2種類を利用して行うことができる

### 古いメッセージを削除する
蓄積されたメッセージの内、古いものから削除する。
削除のトリガーについてはメッセージ取得からの時間経過、データサイズの2種類の設定が可能
- 取得からの時間経過: 時間、分、ミリ秒などで指定可能で、指定の時間より古くなったデータが削除される(デフォルトは1週間)
- データサイズ: 蓄積されたデータが指定したサイズより大きくなった場合にデータが削除される(デフォルトはサイズ制限なし)
上記設定に応じた時刻やデータサイズに達したタイミングで古いデータが削除され、削除されたデータの再取得は不可能となる。

### コンパクション
最新のkeyのデータを残し、重複するkeyの古いメッセージが削除される。
利用例)
RDBMSのINSERTやUPDATEされるレコードをkafkaでも受信するという場合を想定。
UPDATEにより行変更された場合、UPDATE後の最新の値のみ取得できれば良い、というケースではコンパクションの設定にすることで、ディスク容量や入出力を効率的に利用しつつ、各keyの最新あるいは最新を含む一定のレコードを保持し続けられる。


# 参考資料
- [Apache Kafka 分散メッセージングシステムの構築と活用](https://www.amazon.co.jp/Apache-Kafka-%E5%88%86%E6%95%A3%E3%83%A1%E3%83%83%E3%82%BB%E3%83%BC%E3%82%B8%E3%83%B3%E3%82%B0%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%81%AE%E6%A7%8B%E7%AF%89%E3%81%A8%E6%B4%BB%E7%94%A8-NEXT-ONE/dp/4798152374/ref=tmm_pap_swatch_0?_encoding=UTF8&qid=&sr=)
    - 以前前半部分をまとめたものをarabikiさんからもらっていた
- [Apache Kafkaの概要とアーキテクチャ Qiita](https://qiita.com/sigmalist/items/5a26ab519cbdf1e07af3)
