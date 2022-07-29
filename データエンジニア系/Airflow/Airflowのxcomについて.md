# what
- airflowのXComsについて調べたことや実際にやってみた事を記載する


# [XComs](https://airflow.apache.org/docs/stable/concepts.html?highlight=xcom#xcoms) とは
-  airflowに搭載されている機能の1つで、**task間のデータの受け渡しを行う**際に利用する
- variablesと近い印象だが、あちらは「グローバル設定」に対してxcomsは「task間通信用」になっている事に注意
>  Note that XComs are similar to Variables, but are specifically designed for inter-task communication rather than global settings.



## 使い方
 - データを送るときは`xcom_push()`
 - データを受け取るときは`xcom_pull()`
 - 実際に使い方を知ったほうが早そう

## 使ってみる
- 公式のgithubにサンプル用のスクリプトがあるのでこれを利用してみる
    - [example_xcom.py](https://github.com/apache/airflow/blob/master/airflow/example_dags/example_xcom.py)
- 今回は受け渡したときの値が確認できれば良いので上記をもっと簡単にしたスクリプトを作る

### テスト用コード

```python
import datetime
import os

import airflow
# テストがてら追加↓
import logging
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'Data Analysis',
    'depends_on_past': False,
    'email': [''],
    'email_on_failure': False,
    'email_on_retry': False,
    'execution_timeout': datetime.timedelta(hours=1),
    'retries': 3,
    'retry_delay': datetime.timedelta(minutes=5),
    'start_date': datetime.datetime(2019,10,6),
    'on_failure_callback':send_slack_error,
    'provide_context': True
}

with airflow.DAG(
        'TEST_XCOM',
        'catchup=False',
        default_args=default_args,
        schedule_interval='0 23 * * *') as dag:

    def push(**kwargs):
        value_1 = [1, 2, 3]
        # returnした値をpullerに受け渡す
        return value_1


    def puller(**kwargs):
        ti = kwargs['ti']
        # get value_1
        pulled_value_1 = ti.xcom_pull(task_ids='test_push')
        logging.info("pull--> %s", pulled_value_1)


    push_task = PythonOperator(
        task_id = 'test_push',
        python_callable=push,
    )

    pull_task = PythonOperator(
        task_id = 'test_pull',
        python_callable=puller
    )
    pull_task << push_task
```

### log結果(一部抜粋)

```python
[2019-12-11 02:59:45,963] {base_task_runner.py:101} INFO - Job 29649: Subtask test_pull AIRFLOW_CTX_DAG_ID=TEST_XCOM
[2019-12-11 02:59:45,963] {base_task_runner.py:101} INFO - Job 29649: Subtask test_pull AIRFLOW_CTX_TASK_ID=test_pull
[2019-12-11 02:59:45,963] {base_task_runner.py:101} INFO - Job 29649: Subtask test_pull AIRFLOW_CTX_EXECUTION_DATE=2019-12-11T02:58:54.089519+00:00
[2019-12-11 02:59:45,964] {base_task_runner.py:101} INFO - Job 29649: Subtask test_pull AIRFLOW_CTX_DAG_RUN_ID=manual__2019-12-11T02:58:54.089519+00:00
[2019-12-11 02:59:45,986] {base_task_runner.py:101} INFO - Job 29649: Subtask test_pull [2019-12-11 02:59:45,984] {cv_user_annotaions_dag.py:66} INFO - pull--> [1, 2, 3]
```

## 利用の際の注意点
- task_id間違えていると受け渡しできないので注意する
