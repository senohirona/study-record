# what
- AWS Data WanglerでpythonスクリプトからAWS Athenaにクエリできるかを試してみたログ


# AWS Data Wanglerとは
- 公式: https://aws.amazon.com/jp/blogs/news/launch-aws-data-wrangler-v1-0/
- GitHub: https://github.com/awslabs/aws-data-wrangler

# 今回試してみること
 - AWS Data Wanglerを利用してAthenaにクエリを投げて結果が返ってくること


### 利用するもの
- 検証用のAWSアカウント
- S3バケット
- 検証用データ(CSVとか)
- pythonスクリプト
    - poetryを利用して開発環境を構築
    - リポジトリはscriptsリポジトリ内

## 手順
- Athenaに今回検証に使うテーブルを作成する
- pythonでData Wanglerを利用してathenaにクエリを投げるスクリプトを作る
- スクリプトを実行して結果が帰ってくるのか確認

# 準備
## AWS Data Warglerインストール
今回はpoetryでプロジェクトを立ち上げ、その上でスクリプトを書いたり実行したりすることを行う

```zsh
$ poetry add awswrangler
Using version ^2.12.1 for awswrangler

Updating dependencies
Resolving dependencies... (0.0s)

  SolverProblemError

  The current project's Python requirement (>=3.8,<4.0) is not compatible with some of the required packages Python requirement:
    - awswrangler requires Python >=3.6.2,<3.10, so it will not be satisfied for Python >=3.10,<4.0

  Because awswrangler (2.12.1) requires Python >=3.6.2,<3.10
   and no versions of awswrangler match >2.12.1,<3.0.0, awswrangler is forbidden.
  So, because report-scripts depends on awswrangler (^2.12.1), version solving failed.

  at /usr/local/Cellar/poetry/1.1.11/libexec/lib/python3.9/site-packages/poetry/puzzle/solver.py:241 in _solve
      237│             packages = result.packages
      238│         except OverrideNeeded as e:
      239│             return self.solve_in_compatibility_mode(e.overrides, use_latest=use_latest)
      240│         except SolveFailure as e:
    → 241│             raise SolverProblemError(e)
      242│
      243│         results = dict(
      244│             depth_first_search(
      245│                 PackageNode(self._package, packages), aggregate_package_nodes

  • Check your dependencies Python requirement: The Python requirement can be specified via the `python` or `markers` properties

    For awswrangler, a possible solution would be to set the `python` property to ">=3.8,<3.10"

    https://python-poetry.org/docs/dependency-specification/#python-restricted-dependencies,
    https://python-poetry.org/docs/dependency-specification/#using-environment-markers
```

コケた

どうやら、AWS Data Wargler側で指定しているpythonのバージョンとpoetryで設定していたpythonバージョンが合っていたなかった模様
(3.8は指定していたが、それではダメだった模様)
エラー分の中に解決方法が書いてあったので、poetry側のpythonの設定を以下のように書き換える

```
# 変更前
python = "^3.8"
# 変更後
python = ">=3.8,<3.10"
```

再度インストール

```zsh
$ poetry add awswrangler
Using version ^2.12.1 for awswrangler

Updating dependencies
Resolving dependencies... (7.2s)

Writing lock file

Package operations: 28 installs, 0 updates, 0 removals

  • Installing jmespath (0.10.0)
  • Installing urllib3 (1.26.7)
  • Installing botocore (1.22.3)
  • Installing asn1crypto (1.4.0)
  • Installing certifi (2021.10.8)
  • Installing charset-normalizer (2.0.7)
  • Installing idna (3.3)
  • Installing s3transfer (0.5.0)
  • Installing soupsieve (2.2.1)
  • Installing beautifulsoup4 (4.10.0)
  • Installing boto3 (1.19.3)
  • Installing decorator (5.1.0)
  • Installing et-xmlfile (1.1.0)
  • Installing lxml (4.6.3)
  • Installing ply (3.11)
  • Installing python-utils (2.5.6)
  • Installing requests (2.26.0)
  • Installing scramp (1.4.1)
  • Installing jsonpath-ng (1.5.3)
  • Installing openpyxl (3.0.9)
  • Installing opensearch-py (1.0.0)
  • Installing pg8000 (1.21.3)
  • Installing progressbar2 (3.55.0)
  • Installing pyarrow (5.0.0)
  • Installing pymysql (1.0.2)
  • Installing redshift-connector (2.0.889)
  • Installing requests-aws4auth (1.1.1)
  • Installing awswrangler (2.12.1)
```

インストールできた


# Athenaにテーブルを作る
今回はS3バケットにテストデータの入ったCSVファイルを置き、そこから作成する手順をする
(Athenaでテーブル作ったことなかったので)

テストデータは以下のものを利用
[男女別人口－全国，都道府県（大正９年～平成27年）](https://www.e-stat.go.jp/stat-search/files?page=1&layout=datalist&toukei=00200521&tstat=000001011777&cycle=0&tclass1=000001094741&stat_infid=000031524010&tclass2val=0)

### S3バケットにCSVファイルを置く
AWSアカウント上に作成する

S3バケットは新規作成
詳細設定などは特にはしない

作成したバケットにCSVファイルを置く

### Athenaにテーブルを作成する
S3にアップロードしたCSVファイルを利用してテーブルを作成する
初のテーブル作成なので今回はコンソールからやってみる

1. Athenaコンソール画面の左のリストから「テーブルの作成」を選択
2. 「S3から作成」を選択
3. 「テーブルの追加」画面から必要設定を入力していく、そして「次へ」を選択
 データベース: 新しいデータベースを作成 を選択し作成する
テーブル名: toukei-data(CSVのデータが統計情報なので)
入力データセットの場所: 先程作成したS3バケットのURL(S3のページからS3URIを調べられる)
4.「ステップ2:データ形式」でデータの形式は「CSV」を選択
5.  「ステップ3: 列」で列名と型を入力する
今回は以下
- code: string
- prefecture: string
- era_name: string
- japanese_calender: int
- year: int
- comment: string
- all_population: bigint
- man_population: bigint
- woman_population: bigint
6. 「ステップ4:パーティション」については今回は追加はせずに「テーブルの作成」を設定する
作成ができると↓のような画面が出てくる
<img width="1359" alt="Screen Shot 2021-11-01 at 15.02.23.png (327.3 kB)" src="https://img.esa.io/uploads/production/attachments/2285/2021/11/01/25007/aa1c3cf2-2265-4d81-a64b-6811c06ae0c1.png">

# pythonでAthenaにクエリを叩くスクリプトを作る
Data Wanglerを利用してスクリプトを書く

### メモ
- scriptsリポジトリ内では`.envrc`の設定が優先されるため、awscliのcredentialのデフォルト設定を検証用アカウントにしていても`.envcrc`での設定が本番環境のアカウントになっていれば、`.envrc`の設定が優先して実行される
    - 今回`.envrc`のAWSアカウント設定を検証用アカウントのものに変更した
- envchainをかませて実行すれば↑の点は気にしなくてもよい

## スクリプト
とくに複雑な処理はせず、ただAthenaに接続してクエリを実行するだけ
(pandas呼び出しているけれどいらなかったかもしれない)
```python
import pandas as pd
import awswrangler as wr

dfs = wr.athena.read_sql_query(
    "SELECT * FROM test_table.toukei_data limit 10",
    database="test_table",
)

print("session")
print(dfs)
```

### 結果

```zsh
$ poetry run python wrangler-test.py
session
        code prefecture era_name  japanese_calender  year comment  all_population  man_population  woman_population
0  "都道府県コード"    "都道府県名"     "元号"               <NA>  <NA>     "注"            <NA>            <NA>              <NA>
1       "00"       "全国"     "大正"                  9  1920      ""        55963053        28044185          27918868
2       "01"      "北海道"     "大正"                  9  1920      ""         2359183         1244322           1114861
3       "02"      "青森県"     "大正"                  9  1920      ""          756454          381293            375161
4       "03"      "岩手県"     "大正"                  9  1920      ""          845540          421069            424471
5       "04"      "宮城県"     "大正"                  9  1920      ""          961768          485309            476459
6       "05"      "秋田県"     "大正"                  9  1920      ""          898537          453682            444855
7       "06"      "山形県"     "大正"                  9  1920      ""          968925          478328            490597
8       "07"      "福島県"     "大正"                  9  1920      ""         1362750          673525            689225
9       "08"      "茨城県"     "大正"                  9  1920      ""         1350400          662128            688272
```

結果を取得することができた


# 参考資料とか
- https://amagaeru1123.hatenablog.jp/entry/2020/12/15/212014
