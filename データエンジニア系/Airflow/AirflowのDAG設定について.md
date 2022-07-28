# what

- AirflowにおけるDAGの書き方について調べたことを記載する

# why
- 軽くAirflowを触ってみたが、DAGの詳しい書き方についてわからない点が多いため

# 参考文章やコード
- [airflow公式チュートリアル](https://airflow.apache.org/tutorial.html#example-pipeline-definition)

# 書き方

- 大まかにはこんな流れ(状況によって変わる)
    - モジュールのインポート
    - デフォルト引数
    - DAGのインスタンス化
    - タスク

**お手本コード**

<details><summary>長いので折りたたみ</summary>

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta


default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2015, 6, 1),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
}

dag = DAG('tutorial', default_args=default_args, schedule_interval=timedelta(days=1))

# t1, t2 and t3 are examples of tasks created by instantiating operators
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag)

templated_command = """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
        echo "{{ params.my_param }}"
    {% endfor %}
"""

t3 = BashOperator(
    task_id='templated',
    bash_command=templated_command,
    params={'my_param': 'Parameter I passed in'},
    dag=dag)

t2.set_upstream(t1)
t3.set_upstream(t1)

```
</details>

## モジュールのインポート
- pythonで言うimportと意味は同じ
- AirflowのDAGを設定するために必要なモジュールをインポートする

```python
# The DAG object; we'll need this to instantiate a DAG
from airflow import DAG

# Operators; we need this to operate!
from airflow.operators.bash_operator import BashOperator
```

## デフォルト引数
- DAGにはタスク内容を書いていくが、共通している部分や変数についてはデフォルトの引数を設定して呼び出すことができる
- デフォルト引数はそれらを設定する所

```python
from datetime import datetime, timedelta

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2015, 6, 1),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
}
```

## DAG作成
- DAGを作成する。
- ここではdag_idを定義する文字列を渡し、DAGの一意の識別子として機能させる。

```python
dag = DAG(
    'tutorial', default_args=default_args, schedule_interval=timedelta(days=1))
```

## タスク作成
- 実行する処理分タスクを作成する

```python
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag)
```

## タスクの順番を決める
- 処理の順番付けをする
- 記載しているのは一例

```python
# The bit shift operator can also be
# used to chain operations:
t1 >> t2

# And the upstream dependency with the
# bit shift operator:
t2 << t1

# Chaining multiple dependencies becomes
# concise with the bit shift operator:
t1 >> t2 >> t3
```
