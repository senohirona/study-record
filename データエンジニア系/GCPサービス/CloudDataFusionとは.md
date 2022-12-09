GCPのCloudDataFusionというサービスを触ってみたログ(2020年)

# what
- cloud data fusionについて調べてみたことや、実際に触ってみたことを記載する


# [cloud data fusion](https://cloud.google.com/data-fusion?hl=ja)とは
- > Cloud Data Fusion は、データ パイプラインを迅速に構築し管理するための、フルマネージドかつクラウド ネイティブなエンタープライズ データ統合サービスです。(公式より)
    - [ドキュメント](https://cloud.google.com/data-fusion/docs/concepts/overview?hl=ja)
- 操作をブラウザ上で行えるため、コードを書く必要がない
    - コードが書けない人でも直感的に操作が可能


# 実際に触ってみる(クイックスタート編)
- とりあえずクイックスタートやってみる
- [クイックスタートガイド](https://cloud.google.com/data-fusion/docs/quickstart?hl=ja)

### 作業環境
- project
    - 個人のプロジェクト
- リージョン
    - us-west-1

## cloud data fusion APIを有効にする　※初使用時のみ
- data fusionを利用するにはcloud data fusion APIを有効にする必要がある


## インスタンスを作成
- よしなに名前をつけて作成

## ガイドに従って作業
- ガイド用のサンプルパイプラインを用意してあるのでそれを利用
- サンプルパイプラインの流れは以下
1. NYT ベストセラー データを含む JSON ファイルを Cloud Storage から読み取る。
1. ファイルに対し変換を実施して、データの解析とクリーニングを行う。
1. 先週追加された高評価の書籍で $25 未満のものを BigQuery に読み込む。

# 削除作業
- 動作確認後、インスタンス等の削除作業を行う


# 実際に触ってみる(チュートリアル編)
- クイックスタートでは詳しい操作方法などはなかったため、チュートリアルを触ってみる
    - [チュートリアル](https://cloud.google.com/data-fusion/docs/tutorials)
- チュートリアルは複数種類あり、今回はそのうちの「ターゲティング キャンペーン パイプライン」をやってみる
    - [ターゲティング キャンペーン パイプライン](https://cloud.google.com/data-fusion/docs/tutorials/targeting-campaign-pipeline)

## 問題発生
- パイプラインの設定を行い、実際にdeploy->runしてみたところ、以下のエラーが出てきてしまい実行ができなかった
    - 該当のサービスアカウント等に強めの権限を付与してみたが結果はわからずだった
- サンプルとして提供されているバケット側でなにか制限が入ったのかもしれない?

```
932194022375-compute@developer.gserviceaccount.com does not have storage.buckets.get access to the Google Cloud Storage bucket.

```

# 実際に触ってみる(仕切り直し編)
- チュートリアルをそのまま進めることができなかったので、「チュートリアルでの流れを参考にして、独自にパイプラインを作成」して実行してみることにした
    - エクスポートした設定ファイル(json)
        - [test-csv-bq_v1-cdap-data-pipeline.json (7.4 kB)](https://esa-storage-tokyo.s3-ap-northeast-1.amazonaws.com/uploads/production/attachments/2285/2020/06/04/25007/a9bb8b92-1047-4166-b252-d4f7ba140e43.json)


## 利用環境
- GCPプロジェクト
- GCS
- CSVファイル
    - 今回のためのテストファイル


## パイプラインの流れ
- GCSbucketに置いてあるCSVファイルのデータを読み込む
- 該当データを「,」区切りでカラムに分ける
- データをBQにimportする
    - この時、BQテーブルは自動的に作られるように設定する

## 結果
- 無事にGCSからデータを取得し、BQに出力することができた
    - サービスアカウント周りの権限問題も特に起こることはなかった
- BQ tableはdata fusion側で自動的に作成されることも確認できた

# 実際に触ってみる(色々)
**雑多に書いてしまったので、項目ごと整理する予定**
- 確認したい機能がEnterprise エディションでしか利用できない等の関連で、新しくインスタンスを設定して検証してみる

## スケジュール実行
- スケジュール実行機能、タイムゾーンはAirflowと同様UTCな様子
    - 試しに1つ設定して様子を見てみる
- 実行方法についてドキュメントがなかったため以下サイトを参考に操作している
    - [Google Data Fusionパイプラインのスケジュール方法](http://www.366service.com/jp/qa/d7ac53dea590fbc2a4f153283acd2ff0)

### 結果
- スケジュール実行機能は問題なく実行されることが確認できた

## HTTPS source plungin を試してみる
- 現時点(2020年)ではdata fusionにAPIを叩く機能はない。しかし、以下を試してみると可能なのでは?と話題が上がった
- >data fusionにHTTP source pluginがあるらしい https://github.com/cdapio/hydrator-plugins/blob/release/2.3/spark-plugins/docs/HTTPPoller-streamingsource.md
- plugin導入して試してみる

### plugin の導入
- UI画面からpluginの導入は可能だったので、該当のpluginを入れてみようと思ったが、「追加できるplugin」の中に見つからなかった
- google公式では「利用できる」と記載されていた
    - [data](https://cloud.google.com/data-fusion/plugins?hl=ja#/Plugin%20Type=Source)

# 雑多メモ
- **パイプラインの実行はDataproc上で行われるため、パイプラインはDataprocとして課金される**
    - その他、パイプラインで連携しているサービスの実行料が別途加算される
    - 利用要件によっては月に30万超える場合もありそう
- すでにdeployしてしまったパイプラインについては編集ができない
    - [How to edit an already published Cloud Data Fusion Pipeline](https://stackoverflow.com/questions/57606168/how-to-edit-an-already-published-cloud-data-fusion-pipeline)
    - 複製して編集をするという事は可能
- deployしていないパイプラインは「draft」扱いとなり、編集等は可能
- インスタンスにはBasic エディションと Enterprise エディションがあり、スケジュール実行するにはEnterprise エディションを選択しないといけない
    - [Basic エディションと Enterprise エディションの比較](https://cloud.google.com/data-fusion/pricing#comparison_of_basic_and_enterprise_editions)
- フィルターやデータ型変換のカスタマイズについてはスプレッドシートの関数とほぼ同様の記載法
