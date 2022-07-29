# what
- Airflow(正確にはcloud_composerも)を利用してredashのクエリAPIをGCSに送れるようにするために調べたこと、実際にやってみたことを記載する

# やりたいこと
- redashのクエリAPIを実行し、取得した結果をGCSにアップロードする

## 実行する環境
- google cloud composer
    - Airflow
        - python3
    - 実験用bucket(今回は自分で用意していたものを使用)

# やってみたこと
## redashのクエリAPIがAirflowで実行可能か確認
### 手段その1
- curlコマンドを利用してAPIを叩く
    - bash_operatorを利用
- 実行した際に下記のエラーが出た
    - `ERROR - 'utf-8' codec can't decode bytes in position 40-41: invalid continuation byte`
    - python側のエラーではあるが、curlの結果を変数に入れることで解決

### 手段その2
- pythonの「requests」を利用してAPIを叩く
    - [Requests: HTTP for Humans](https://pypi.org/project/requests/)
- 別途ライブラリをインストールする可能性があったが、Composerでライブラリを追加しなくとも利用できた

## 取得したデータをGCSに送る
- 「google-cloud-storage」というライブラリを利用してみた
    - [google-cloud-storage](https://pypi.org/project/google-cloud-storage/)
- composerへはコンソール画面からインストールを行った
    - [PyPi から Python 依存関係をインストールする](https://cloud.google.com/composer/docs/how-to/using/installing-python-dependencies?hl=ja)
- 結果、用意したbucketにファイルを追加することができた
- また、追加したファイルを元にBQでテーブルを作成することができる事を確認した
    - GCSにおけるファイルタイプが「text/plain」ではあるがデータの欠損や文字化け等はなかった


## 上記2つを合わせた結果
#### テキスト
- 変数名等、ライブラリの例を参考に記載しているので見づらいかもしれないです

```python
import datetime
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
from google.cloud import storage
import requests


client = storage.Client()


dag = DAG(
    'test_dag',
    schedule_interval = '@daily',
    start_date=datetime.datetime(2019, 8,3)
)

def redash_to_gcs(ds, **kwargs):
    result = requests.get('取得したいredashのクエリURL')
    print(result)
    bucket = client.get_bucket('送信先バケット')
    blob = bucket.blob("result_files.csv")
    blob.upload_from_string(result.text.encode('utf-8'))

to_gcs = PythonOperator(
    task_id='echo_results',
    python_callable=redash_to_gcs,
    provide_context=True,
    dag=dag
)
```

# 疑問点・調べたこと
## 調べたこと
### Airflowを利用してGCSにファイルやデータを送る方法について
- curl(REST API) を利用する方法
- クラウドストレージクライアントライブラリを利用してpythonでスクリプト書く
    - [オブジェクトのアップロード](https://cloud.google.com/storage/docs/uploading-objects#storage-upload-object-python)
- googlecloudstoragehookを利用する
    - ただし、まだ実装されていないらしく別途プラグインをインストールする必要がある
    - [公式ドキュメント](https://airflow.apache.org/_api/airflow/contrib/hooks/gcs_hook/index.html?highlight=googlecloudstoragehook#module-airflow.contrib.hooks.gcs_hook)
- google-cloud-storageを利用する
    - こちらはpythonライブラリ
    - テキスト等もGCSにアップロードできるらしい
    - [ドキュメント](https://pypi.org/project/google-cloud-storage/)
    - [<関連>ストリーミング転送](https://cloud.google.com/storage/docs/streaming)

### composerに必要ライブラリをインストールする方法
- 公式ドキュメントに載っていた
    - [Python 依存関係のインストール](https://cloud.google.com/composer/docs/how-to/using/installing-python-dependencies?hl=ja)
