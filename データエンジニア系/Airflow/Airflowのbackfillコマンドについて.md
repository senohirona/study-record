# what
- airflowのbackfillコマンドについて、概要と基本的な使い方を記載する

# backfillコマンドとは
- airflowに存在するサブコマンドの一つで、「過去の期間を指定してdagを実行する」コマンド
    - [backfill](https://airflow.apache.org/docs/stable/cli.html#backfill)
- 期間やタスクを指定して実行することが可能

## オプション(最低限のものだけ)

- -t
    - 実行するタスクの指定
- -s
    - 期間の開始日
- -e
    - 期間の終了日
    - ここで指定した日付は**実行期間の対象外となる点に注意
- -m
    - 実行結果に無条件でsuccessフラグを付与する

## 注意点
- 長期間でbackfillを実行すると、まれにretryが起こる場合がある
    - `'Task Instance Slots Available' FAILED: The maximum number of running tasks (30) for this task's DAG 'BigQuery' has been reached.`
    - その際は、retryしているdagrunを一旦clearしてbackfillをやり直してみると良い
- 複数のdagでbackfillを実行していると、airflow側でデッドロックが起こることがある
    - `Deadlock found when trying to get lock; try restarting transaction`
    - この場合は、dagを1つ1つ実行するようにする
