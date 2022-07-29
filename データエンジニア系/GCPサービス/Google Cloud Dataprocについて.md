# what
- google cloud dataroc(dataproc)について基本概要や調べてわかったことを記載する


# [Dataproc](https://cloud.google.com/dataproc/docs/concepts/overview?hl=ja)とは
- GCP 上で手軽に Hadoop や Spark のクラスターを立ち上げ、データの解析を行うサービス
    - > オープンソースのデータツールを利用してバッチ処理、クエリ実行、ストリーミング、機械学習を行えるマネージド Spark / Hadoop サービスです
- Hadoopやsparkは利用するための環境構築等、準備が大変な面があった
    - しかしDataprocを利用すれば、複雑なことを考えずHadoopやSparkを利用できるというメリットがある


## Hadoopとかsparkについて
- HadoopやSparkは**分散処理フレームワーク**と呼ばれる
    - 意味合い的には「**大量のデータを高速で処理できる(分散処理)ようにする機能をシステムに組み込む雛形(フレームワーク)**」か?
- おたがいに長所短所は存在するので「目的に応じて」使い分けるのが正解らしい
    - 「**リアルタイムの高速処理が求められるデータはSparkで、メモリに乗り切る以上のサイズのデータを処理する場合はHadoopで**」と言われているとか

## Hadoop
- Apachプロジェクトに登録されている「多量のコンピューターで大量のデータ処理を行うためのシステム」(OSS)
    - [公式](https://hadoop.apache.org/)
- データをHDDやSSDに入れて処理するため、膨大な量のデータを処理することが可能
- リアルタイムで特定のデータを見つけ出すような高速処理には向かない、という欠点が存在する
- HiveはクエリをHadoop上で実行するためのソフトウェア

## spark
- Hadoopと同じくApachプロジェクトに登録されているOSS
- データをメモリに入れて処理するため、高速処理に向いている
- 「メモリに乗り切る」データ量で処理するので、膨大なデータの処理には向かない


# 参考資料
 - [【GCP入門編・第11回】 Google Cloud Dataproc を使ってデータを解析しよう！](https://www.topgate.co.jp/gcp11-analyze-data-using-google-cloud-dataproc)
 - [Apache HadoopとSparkの違いとは？ 分散処理フレームワークを基礎からわかりやすく解説！](https://data.wingarc.com/hadoop_spark-20912)
