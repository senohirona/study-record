# what
- airflowのbaseoperatorにおけるcallbackとtrigger_ruleについて記載する
    - [airflow.models.baseoperator](https://airflow.apache.org/_api/airflow/models/baseoperator/index.html?highlight=on_failure_callback)


# callback
- ここで記載するcallbackは下記のこと
    - `on_failure_callback`
    - `on_retry_callback`
    - `on_success_callback`

## on_failure_callback
 - taskが失敗した際に呼び出される関数

## on_retry_callback
- taskが再実行(retry)した際に呼び出される関数

## on_success_callback
- taskが成功した際に呼び出される関数


# trigger_rule
- task複数存在し依存関係がある時、taskを実行する際のルールを定義する
    - [Trigger Rules  Airflow](http://airflow.apache.org/concepts.html?highlight=trigger#trigger-rules)
