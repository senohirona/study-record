# aws ecs(awscli)コマンドとは

- そもそもawscliについてもざっくりとしか知らないので、これを機に調べてみる

## awscli(AWS コマンドラインインターフェイス)とは

- awsサービスをコマンドラインで操作するツール
    - webコンソールで設定する部分を自動化する際などに利用される
- すべてのサービス(RDS,ECS等)で利用ができる(サービスが多いので、覚えるのは大変な気がする)
- 下記公式概要
> AWS コマンドラインインターフェイス (CLI) は、AWS サービスを管理するための統合ツールです。ダウンロードおよび設定用の単一のツールのみを使用して、コマンドラインから複数の AWS サービスを制御し、スクリプトを使用してこれらを自動化することができます。

- [公式によるサックリとした概要](https://aws.amazon.com/jp/cli/)
- [公式ドキュメント](https://docs.aws.amazon.com/ja_jp/cli/index.html#lang/ja_jp)


## aws ecsコマンドとは

- awscliでecsに対して操作を加える際に利用するコマンド
    - [公式ドキュメント](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ECS_AWSCLI.html)
    - [ecsのコマンド一覧](https://docs.aws.amazon.com/cli/latest/reference/ecs/index.html#cli-aws-ecs)

## コマンドの使い方
- 今回は、利用する下記について調べる
    - update-service
    - run-task

## update-service
- サービスのパラメータを変更する
    - [コマンドリファレンス](https://docs.aws.amazon.com/cli/latest/reference/ecs/update-service.html)
- タスクの数は`desired-count`という引数で変更することができる

### 使用例

- サービス内のタスクの数を10に変更する

```
aws ecs update-service --service my-http-service --desired-count 10
```

- 利用されている引数について解説
    - `--service`: 更新するサービスの名称を記載する
    - `--desired-count`:サービスのタスクの数を記載する


## run-task
- 指定されたタスク定義を使用して新しいタスクを開始する
    - [コマンドリファレンス](https://docs.aws.amazon.com/cli/latest/reference/ecs/run-task.html)
- DBのmigrate(コマンドの上書き)は`--overrides`
