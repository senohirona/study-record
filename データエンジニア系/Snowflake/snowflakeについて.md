# 概要

[クラウドデータプラットフォーム| Snowflake 【スノーフレイク】](https://www.snowflake.com/snowflake-cloud-data-platform/?lang=ja)

```sql
無限の可能性を持つシームレスなデータコラボレーションにより、すべてのデータと基本ワークロードを管理するグローバルプラットフォーム
```

- Snowflake社の提供するクラウド上で動くSaas型のデータウェアハウス(DWH)
- 類似ツールとしては以下が代表的
    - Amazon Redshift
    - Google BigQuery

## 基本構造

snowflakeは3層のレイヤーから構成されている。

- Data Storage Layer
- Query Processing Layer
- Cloud Service Layer
****[重要な概念およびアーキテクチャ—snowflake](https://docs.snowflake.com/ja/user-guide/intro-key-concepts.html)****

### Cloud Service Layer

snowflakeの各種管理を担うレイヤー

認証認可、メタデータ管理を行うほか、クエリを実行前に最適化したりプルーニングしたりと言ったSQLオプティマイザや、トランザクション管理を全般的に処理する。

### Query Processing Layer

仮想ウェアハウスが稼働するレイヤー。

(仮想ウェアハウス: データに対してクエリを実行するマシンのこと。それぞれにCPU・メモリ・SSDが備わっている。SQLなどの実行時に稼働する)

仮想ウェアハウスの作成には事実上制限がなく、組織の役割やプロジェクトに応じて自由に作成することができる。

ストレージレイヤーと完全に分離する形で、クエリを実行・操作するレイヤーが存在している。

### **Database Storage Layer**

snowflakeにロードサれた全データが置かれるレイヤー

データはそれぞれ、内部の最適化サれたマイクロパーティションとして保存される。

このレイヤーに置かれたデータに、ユーザーが直接触れることはできない。

# 料金

[料金体系 | Snowflake 【スノーフレイク】](https://www.snowflake.com/pricing/?lang=ja)

AWSやGCPと比較すると若干複雑?かもしれない

## 契約形態

snowflakenの契約形態は大きく2種類用意されている

- オンデマンド
    - 基本的な契約形態
    - リソースを使用した分だけ料金が発生する重課金方式
    - 料金の支払いは毎月
- プリペイドキャパシティ
    - 前払方式
    - 事前に契約した容量から、毎月の使用量分が消費されていく仕組み
    - 契約期間は年単位。
        - 契約容量もしくは期間が終了した後でも引き続き利用は可能だが、その場合はオンデマンドに対応する価格が請求される

## 料金体系とクレジット

snowflakenの料金は、**[ウェアハウス使用料+ストレージ使用料]**で構成される。

その中でも以下5つのポイントを抑えておく必要がある。

- クラウドサービスとリージョン
- エディション
- クレジット
- ウェアハウス使用料
- ストレージ使用料

### クラウドサービスとリージョン

「どのリージョン」で「どのクラウドサービスを利用するか」で料金が変わる。

2023現在、対応しているクラウドサービスは以下。(ただし、東京リージョンに対応しているのはAWSのみ)

- **Amazon Web Service(AWS)**
- **Microsoft Azure**
- **Google Cloud Platform**

### エディション

Snowflakeには４つのエディションが用意されている。

スタンダードエディションが一番基本で、そこからグレードアップしていく形式になっている

- スタンダード: 一番基本のプラン
- エンタープライズ
- ビジネスクリティカル
- バーチャルプライベート

### クレジット

ここでは「Snowflakeのリソース使用量の単位」を意味する。

snowflakeの仮想ウェアハウス（コンピュートリソース）をどの程度使用したのかをクレジットという単位で表し、1クレジットあたりの価格が上記の選択（クラウドサービス、リージョン、エディション）によって定義される。

### ウェアハウス使用料

仮想ウェアハウスをどの程度使用したかを表す。

仮想ウェアハウスは、XS、S、M、L、XLといったサイズが用意されており、サイズによってクエリの実行速度が決まる。

重要なのは、**クレジットが消費されるのは稼働した分に対してのみ**ということ。

**待機時間中にウェアハウスを停止させておけば、その分の費用は発生しない**。

### ****ストレージ使用料****

Snowflakeに保存しているデータ容量のこと。

１ヶ月あたりの平均バイト数に基づいて計算され、以下のデータが対象。

- バルクデータのロード/アンロードのためにステージングされているファイル
- Time Travel用に保持される履歴データを含む、データベーステーブルに保存されたデータ
- Fail-safeのために保持される履歴データ

# 特徴

いくつかあるが、その中から4点記載。

## **SQLやGUIによる分析が可能**

Snowflakeでは、ANSI準拠のSQLが使用可能。

さらに、GUIによるテーブル作成が用意されており、SQLの扱いに慣れていないユーザーがマウスとキーボード操作によりテーブルを作成することも可能となっている。

## 高速なパフォーマンス

各々独立したクラスターが並列で処理しており、負荷に応じてクラスタ数やサイズを最適化できる。

また、複数の層でキャッシュを保持できるため、これを利用してユーザーに結果をすぐに返答することができる。

そして、キャッシュを利用し面倒なチューニングなしに検索を最適化できる。

## ステージを使ったデータロード

snowflakeにデータをロードする際は、データを一旦ステージに配置し、ステージからsnowflakeにロードする形を取る。

ステージとは、showflake用のファイルリポジトリのことで、snowflakeにとってデータをロード・アンロードしやすい環境となっている。

これには2種類存在する

- Internal Storage(内部ストレージ): snowflakeの内部に作るストレージ
- External Storage(外部ストレージ): snowflakeの外に作るストレージ

External Storageとして扱えるクラウドストレージは以下の3つになる

- Amazon S3
- Azure Blob
- Google Cloud Storage

## 過去データにアクセス

snowflakeにはタイムトラベルという機能があり、過去データにアクセスが可能。

データベース、テーブル、スキーマといった各種過去データに対して、クエリの実行、複製、リストアが可能。遡れす日数は契約しているエディションによって異なる。

StandardとPremiumでは最大1日、Enterprise以上では最大90日遡ることが可能。

また、タイムトラベル機能の有効期間を過ぎたデータに対して、さらに7日間保持するフェールセーフ機能も備えている。

ただし、フェールセーフ機能はSnowflake社の担当者のみがアクセス可能であり、復元には特殊な手続きが必要な点に注意。

# セキュリティ

[セキュリティ機能の概要 — Snowflake Documentation](https://docs.snowflake.com/ja/user-guide/admin-security.html)

| カテゴリ | 機能 | 利用可能なEdition |
| --- | --- | --- |
| ネットワーク/サイトへのアクセス | IP 許可リストとブロックリストによって制御されるサイトアクセスは、 https://docs.snowflake.com/ja/user-guide/network-policies.htmlによって管理されます。 | すべて |
|  | セッションポリシー を介した、アカウントまたはユーザーの構成可能なアイドルセッションタイムアウト。 | Enterprise Edition |
|  | VPC/VNet と https://docs.snowflake.com/ja/user-guide/private-snowflake-service.html
 の間のプライベート通信 | Business Critical Edition |
|  | https://docs.snowflake.com/ja/user-guide/private-internal-stages.htmlへのプライベート通信。 | Business Critical Edition |
| ユーザーおよびグループ管理 | https://docs.snowflake.com/ja/user-guide/scim.html ユーザーIDとグループ（つまり、ロール）を管理します。 | すべて |
| アカウント/ユーザー認証 | クライアント認証によるセキュリティ強化のための https://docs.snowflake.com/ja/user-guide/key-pair-auth.html
。 | すべて |
|  | https://docs.snowflake.com/ja/user-guide/security-mfa.html（多要素認証）ユーザーによるアカウントアクセスのセキュリティ強化。 | すべて |
|  | https://docs.snowflake.com/ja/user-guide/oauth.htmlユーザーのログイン認証情報を共有または保存せずに認証されたアカウントにアクセス。 | すべて |
|  | フェデレーション認証によるユーザー https://docs.snowflake.com/ja/user-guide/admin-security-fed-auth.html（シングルサインオン）のサポート。 | すべて |
|  | 複数のアクティブなキーをサポートするための基本認証（つまり、ユーザー名とパスワード）およびキーペアローテーションの代替としての https://docs.snowflake.com/ja/user-guide/key-pair-auth.html。 | すべて |
| オブジェクトセキュリティ | DAC （随意アクセス制御）と RBAC （ロールベースのアクセス制御）のハイブリッドモデルを経由した、アカウントに含まれる https://docs.snowflake.com/ja/user-guide/security-access-control.html（例: ユーザー、ウェアハウス、データベース、テーブル）。 | すべて |
| データセキュリティ | Snowflakeテーブルに保存されているすべてのインジェスト済みデータは、 AES-256の強力な暗号化を使用して https://docs.snowflake.com/ja/user-guide/security-encryption.htmlされます。 | すべて |
|  | データのロードおよびアンロードのために内部ステージに保存されているすべてのファイルは、 AES-256の強力な暗号化を使用して自動的に暗号化されます。 | すべて |
|  | 暗号化されたデータの https://docs.snowflake.com/ja/user-guide/security-encryption.html。 | Enterprise Edition |
|  | https://docs.snowflake.com/ja/user-guide/security-encryption.html を使用したデータ暗号化のサポート。 | Business Critical Edition |
| セキュリティの検証 | Soc 1タイプ II およびSoc 2タイプ II コンプライアンス。 | すべて |
|  | HIPAA コンプライアンス。 | Business Critical Edition |
|  | HITRUST CSF コンプライアンス | Business Critical Edition |
|  | PCI DSS コンプライアンス。 | Business Critical Edition |
|  | FedRAMP 中程度のコンプライアンス（指定された https://docs.snowflake.com/ja/user-guide/intro-regions.html#label-us-gov-regions内）。 | Business Critical Edition |
|  | IRAP 保護コンプライアンス（指定された https://docs.snowflake.com/ja/user-guide/intro-regions.html#label-asia-pacific-regions内）。 | Business Critical Edition |

# 参考

- [Snowflakeの紹介](https://docs.snowflake.com/ja/user-guide-intro.html)
- [重要な概念およびアーキテクチャ](https://docs.snowflake.com/ja/user-guide/intro-key-concepts.html)
- [公式ドキュメント](https://docs.snowflake.com/ja/index.html)