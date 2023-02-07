# 【調査】AWS Glueについて

# 概要

AWS Glue は、サーバーレスでスケーラブルなデータ統合サービスで、分析、機械学習、アプリケーション開発用に、複数のソースからデータを検出、準備、移動、統合することを容易にします。

AWS Glue はデータ統合に必要なすべての機能を備えているため、インサイトを得て、数か月どころか数分でデータを使用可能にします。

セットアップまたは管理するインフラストラクチャはありません。ジョブの実行中に使用するリソースに対してのみ支払いが発生します。

(公式ドキュメントより引用)

[AWS Glue（分析用データ抽出、変換、ロード (ETL) ）| AWS](https://aws.amazon.com/jp/glue/)

## 基本構成

4つの要素で構成されている。

- **AWS Glue Data Catalog(データカタログ)**
    - Apache Hiveのメタストアと同じような方法で、AWSクラウド内のメタデータを保存や注釈付け、共有などができる永続的なメタストア
    - 異種のシステムがサイロ化されているデータを追跡しクエリやデータ変換を実施できるよう、メタデータ保管・探索する統合リポジトリとして機能する
- **AWS Glue Crawlers(クローラー)**
    - あらゆる種類のデータをスキャンし、分類やスキーマ情報の抽出、メタデータの保管などを自動で行うクローラー
- **AWS Glue ETL ジョブ**
    - ETL処理用のScalaかPySparkのスクリプトを自動生成
- **Workflow(ワークフロー)**
    - ETLジョブ、クローラ、データカタログ出力までの一連の処理を自動化する

## 周辺機能

AWS Glueは基本構成の他に周辺機能として以下を揃えている。

## [AWS Glue Studio](https://docs.aws.amazon.com/ja_jp/glue/latest/ug/what-is-glue-studio.html)

AWS Glue での抽出、変換、ロード（ETL）ジョブの作成、実行、およびモニタリングを簡単にできるようにする新しいグラフィカルインターフェイス。

データ変換ワークフローを視覚的に作成し、AWS Glue の Apache Spark ベースのサーバーレス ETL エンジン。

現在はこの機能を主軸に展開している。

## [AWS Glue DataBrew](https://docs.aws.amazon.com/databrew/latest/dg/what-is.html)

ユーザーがコードを記述せずにデータをクリーンアップおよび正規化できる視覚的なデータ準備ツール。

データアナリストやデータサイエンティスト向けのデータクリーニングや正規化を行えるようにできているのが特徴。

「Glue Studio」と似ているが、こちらはクリーニングしてデータを整えたり、データソースの変換を視覚的にトラッキングするなど、モデル作成するにあたってのデータ準備向けのものとなっている。

参考: **[【AWS Black Belt Online Seminar】AWS Glue DataBrew](https://d1.awsstatic.com/webinars/jp/pdf/services/20210217_BlackBelt_GlueDataBrew.pdf)**

## [AWS Glue Data Quarity](https://aws.amazon.com/jp/glue/features/data-quality/)

2022年末に登場。**現在プレビューのサービスであるため、変更が入る可能性あり。**

データ品質ルールの作成、管理、モニタリングを自動化し、データレイクやパイプライン全体で高品質なデータを確保できるよう支援する機能。

# サポートデータソース一覧

[AWS Glue: 仕組み](https://docs.aws.amazon.com/ja_jp/glue/latest/dg/how-it-works.html)

### **データソース**

**データソース**: データの抽出元

- データストア
    - Amazon S3
    - Amazon Relational Database Service (Amazon RDS)
    - JDBC アクセスが可能なサードパーティのデータベース
    - Amazon DynamoDB
    - MongoDB と Amazon DocumentDB (MongoDB 互換)
- データストリーム
    - Amazon Kinesis Data Streams
    - Apache Kafka

### データターゲット

**データターゲット**: 変換されたデータがジョブにより書き込まれる場所

(DataFusionでいう「シンク」)

- Amazon S3
- Amazon Relational Database Service (Amazon RDS)
- JDBC アクセスが可能なサードパーティのデータベース
- MongoDB と Amazon DocumentDB (MongoDB 互換)

# 料金

[料金 - AWS Glue | AWS](https://aws.amazon.com/jp/glue/pricing/)

## ETLジョブ及びインタラクティブセッション

使用時間による従量課金制。

```
AWS Glue を使用すると、ETL ジョブの実行に費やした時間に対してのみ支払うことができます。リソースの管理や初期費用は不要で、スタートアップ時間やシャットダウン時間も課金されません。
```

[AWS Glue Pricing | Serverless Data Integration Service | Amazon Web Services](https://aws.amazon.com/jp/glue/pricing/)

ETLジョブには、以下4種類のタイプが存在している。

それぞれ 、DPU時間あたり各リージョンごと(**東京リージョン: $0.44**)の課金額が秒単位で課金される。

- Apache Spark
- Apache Spark Streaming
- Ray(プレビュー)
- Python Shell

**追加料金**

Amazon S3、Amazon RDS、Amazon Redshift などのデータソースからの ETL データの場合は、標準のリクエストおよびデータ転送料金が請求される。

Amazon CloudWatch を使用する場合、CloudWatch Logs および CloudWatch Events の標準料金が請求される。

## データカタログの保存とリクエスト

AWS Glue データカタログを使用すると、最大 100 万個のオブジェクトを無料で保存できる。

保存したオブジェクトの数が 100 万個を超えた場合、100 万個を超えた 100,000 個のオブジェクトごとに毎月 1.00USD(東京リージョン) が請求される。

**東京リージョンにおいての料金**

- **ストレージ**
    - 最初の 100 万個のオブジェクトの保存は無料
    - 月に 100 万個を超えると、10 万個のオブジェクトの保存あたり 1.00USD
- **リクエスト**
    - 最初の 100 万回のリクエストは毎月無料
    - 月に 100 万回を超えると、100 万回のリクエストあたり 1.00USD

## クローラ

AWS Glue クローラランタイムがデータを検出し、AWS Glue データカタログにデータを取り込む場合は、時間あたりの料金がかかる。

**東京リージョンにおいての料金**

- DPU 時間あたり 0.44USD が 1 秒単位で課金され、クローラの実行ごとに最低 10 分

## DataBrewインタラクティブセッション

セッションは、DataBrew プロジェクトのデータがロードされた後に開始。

使用したセッションの総数で課金され、30 分単位で計算される。

**東京リージョンにおいての料金**

- 1.00USD/セッション

## DataBrewジョブ

AWS Glue DataBrew を使用すると、ジョブの実行時にデータのクリーンアップと正規化に使用した時間に対してのみ料金が発生。

ジョブの実行に使用された AWS Glue DataBrew ノードの数に基づいて 1 時間ごとの料金が請求される。

**東京リージョンにおいての料金**

- 0.48USD/DataBrew ノード時間

**追加料金**

AWS Glue DataBrew のジョブが他の AWS サービスを使用する、またはデータを転送すると、追加料金が発生する場合がある。

例)

DataBrew のジョブが Amazon S3 との間でデータを読み書きする場合。

読み取り/書き込み要求について、また Amazon S3 に格納されたデータに対して課金される。

## データ品質

### データ品質タスク

Data Catalog からデータセットを選択し、レコメンデーションを生成。

このアクションは、データ処理ユニット (DPU) をプロビジョニングするためのレコメンデーションタスクを作成します。レコメンデーションを受けたら、ルールを修正したり、新たに追加してスケジュールを組むことができる。

これらのタスクはデータ品質タスクと呼ばれ、DPU をプロビジョニングすることになる。

**東京リージョンにおいての料金**

- DPU 時間あたり 0.44 USD

### **データセットのデータ品質の価格設定または管理**

AWS Glue ETL ジョブにデータ品質チェックを追加して、不良データがデータレイクに入るのを防止することも可能。

これらのデータ品質ルールは ETL ジョブ内に存在するため、実行時間の増加や DPU の消費量の増加を招く。

**東京リージョンにおいての料金**

- DPU 時間あたり 0.44 USD

### 追加料金

AWS Glue は、Amazon Simple Storage Service (Amazon S3) から直接データを処理する。そのため、ストレージ、リクエスト、データ転送に対して Amazon S3 の標準料金が発生する。

利用者の設定に基づき、一時ファイル、データ品質結果、シャッフルファイルは、利用者が選択した S3 バケットに保存され、S3 の標準料金で課金もされる。

データカタログを使用する場合は、標準の AWS Glue データカタログのレートで課金される。

# セキュリティ

[AWS Glue でのセキュリティ](https://docs.aws.amazon.com/ja_jp/glue/latest/dg/security.html)

## **[AWS Glue での Identity and Access Management](https://docs.aws.amazon.com/ja_jp/glue/latest/dg/authentication-and-access-control.html)**

AWS GlueはAWS IAMを使用してアクセス制御を行います。これにより、特定のユーザーまたはグループに対してのみデータアクセスを許可することができます。

## [データの保護](https://docs.aws.amazon.com/ja_jp/glue/latest/dg/data-protection.html)

AWS Glue には、データを保護するために設計された機能がいくつか用意されています。

- 保管中の暗号化
    - [AWS Key Management Service (AWS KMS)](http://aws.amazon.com/kms/) キーを使用して保管時の暗号化されたデータを書き込むように ETL (抽出、変換、ロード) ジョブと開発エンドポイントを設定できます。また、AWS KMS で管理するキーを使用して、[AWS Glue Data Catalog](https://docs.aws.amazon.com/ja_jp/glue/latest/dg/components-overview.html#data-catalog-intro)に格納されているメタデータを暗号化することもできます。さらに、AWS KMS キーを使用して、ジョブのブックマークや、[クローラー](https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html)および ETL ジョブで生成されたログを暗号化することもできます。
- 転送中の暗号化
    - AWS は転送中のデータに対して Transport Layer Security (TLS) 暗号化を提供します。AWS Glue の [[security configurations]](https://docs.aws.amazon.com/glue/latest/dg/console-security-configurations.html)(セキュリティ設定) を使用して、クローラー、ETL ジョブ、開発エンドポイントの暗号化を設定できます。Data Catalog の設定を介して、AWS Glue Data Catalog 暗号化を有効にできます。

## [コンプライアンス検証](https://docs.aws.amazon.com/ja_jp/glue/latest/dg/compliance.html)

AWS Glue を使用する際のお客様のコンプライアンス責任は、お客様のデータの機密性や貴社のコンプライアンス目的、適用可能な法律および規制によって決定されます。AWS Glue の使用が HIPAA、PCI、または FedRAMP などの規格に準拠していることを前提としている場合、AWS は以下を支援するリソースを提供します。

- [セキュリティとコンプライアンスのクイックスタートガイド](http://aws.amazon.com/quickstart/?awsf.quickstart-homepage-filter=categories%23security-identity-compliance)
    - これらのデプロイガイドでは、アーキテクチャ上の考慮事項について説明し、機密性とコンプライアンスに焦点を当てたベースライン環境を AWS にデプロイするためのステップを提供します。
- [Architecting for HIPAA Security and Compliance Whitepaper](https://d0.awsstatic.com/whitepapers/compliance/AWS_HIPAA_Compliance_Whitepaper.pdf) (HIPAA のセキュリティとコンプライアンスのためのアーキテクチャの設計に関するホワイトペーパー)
    - このホワイトペーパーは、企業が AWS を使用して HIPAA 準拠のアプリケーションを作成する方法について説明します。
- [AWS コンプライアンスのリソース](http://aws.amazon.com/compliance/resources/)
    - このワークブックとガイドのコレクションは、お客様の業界や所在地に適用される場合があります。
- [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config.html)
    - この AWS のサービスでは、自社プラクティス、業界ガイドライン、および規制に対するリソースの設定の準拠状態を評価します。
- [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
    - この AWS サービスでは、AWS 内のセキュリティ状態を包括的に表示しており、セキュリティ業界の標準およびベストプラクティスへの準拠を確認するのに役立ちます。

## [ログ記録とモニタリング](https://docs.aws.amazon.com/ja_jp/glue/latest/dg/logging-and-monitoring.html)

ETL (抽出、変換、ロード) ジョブの実行を自動化できます。AWS Glue は、モニタリングできるクローラとジョブのメトリクスを提供します。必要なメタデータを使用して AWS Glue Data Catalog を設定すると、AWS Glue は環境のヘルスに関する統計を提供します。クローラとジョブの呼び出しを、cron に基づく時間ベースのスケジュールで自動化することができます。イベントベースのトリガーが発生したときにジョブをトリガーすることもできます。

Amazon CloudWatch Events を使用して、AWS のサービスを自動化し、アプリケーションの可用性の問題やリソースの変更などのシステムイベントに自動的に応答します。AWS のサービスからのイベントは、ほぼリアルタイムで CloudWatch Events に配信されます。

# 参考

- [AWS Glueの概要](https://docs.aws.amazon.com/ja_jp/glue/latest/dg/what-is-glue.html)
- [よくある質問](https://aws.amazon.com/jp/glue/faqs/)
- [ETLとは(AWSの概要説明)](https://aws.amazon.com/jp/what-is/etl/)