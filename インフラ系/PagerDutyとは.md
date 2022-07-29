# what
- pager dutyについて軽く調べた概要等を記載する


# PagerDutyとは
- PagerDuty Inc.が提供しているインシデント管理サービス
    - [PagerDuty (日本語版)](https://ja.pagerduty.com/)
- 監視ツールやアプリケーションからのアラートを受けて、インシデントの発生を担当者に通知するソフトウェアのこと
- 通知方法は以下がある
    - メール
    - 音声電話
    - SMS
    - プッシュ通知
- PagerDutyは様々なアプリケーションと連携する事が可能。
    - 例
        - slack
        - datadog
        - awsの各サービス(cloudwatch 等) etc...


## PagerDutyでできること
1. 複数のアラートを集計して担当者に通知する。もし、担当者が決められた時間以内に通知確認をしなければ、予め決められた順番で他の担当者に通知する(エスカレーション)
1. 連携できる監視ツールは200種類。REST APIでのアラートも受け付ける。
1. インシデントの状況を対応チーム全体がダッシューボードで確認できる。
1. インシデント対応を予めワークフロー化しておける。
1. オンコールエンジニアのスケジューリングを簡単に行える。
etc...

# incident
PagerDuty側ではインシデントは３つの状態に分けられている
- Triggered
    - インシデントが発生した状態
    - このときに設定された通知先に通知を送る
- Acknowledged
    - インシデントが対応者によって確認し、対応を行っている状態
    - サービスで定義されているAcknowledgement timeoutの時間が経過するまでは、通知が飛ぶことはない
    - 逆を言えば、この時間が経過し多段階で状態がAcknowledgedのままだとTriggeredに戻ってしまい、再度通知が飛んでしまう
- Resolved
    - インシデントが解決した状態
    - この状態になったインシデントは再度通知が行われることもない。ステータスを変更することもできない

# API
PagerDutyには2種類のAPIを提供している
- REST API
- Event API

## REST API
PagerDutyのアカウントにアクセスし、そのアカウントの持つデータなどを操作する事ができるAPI。
インシデントを手動で作成するといった、イベントについても変更やデータを送信する事はできるが、そういったものについては後述するEvent APIのほうが特化している。
現在はv2がメインで利用されている。
[REST APIv2の概要](https://developer.pagerduty.com/docs/rest-api-v2/rest-api/)

APIキーには汎用RESTAPIキーとパーソナルRESTAPIキーが存在している。
汎用RESTAPIキーは一定の権限が付与されているユーザーでなければ生成することができない
パーソナルRESTAPIキーは個人用のRESTAPIキーで、ユーザー側でそれぞれ生成できる。
(ただし、利用できる機能についてはそのユーザーが所持している権限に依存する)


## Event API
サービスのイベントに関する操作を行うためのAPI
v1とv2が存在しており、現在はv2が主流( しかし、一部の機能にはまだ対応していないため、その場合はv1を使うということもある模様)。
[Event API v2の概要](https://develper.pagerduty.com/docs/events-api-v2/overview/)

主な仕様方法には以下が存在する
- インシデントイベントを作成する
- イベントを変更する

### インシデントイベントを作成する
PagerDutyではテスト目的などで手動でインシデントを作成することができるが、Event APIでも作成することができる。
[Send an Alert Event](https://developer.pagerduty.com/docs/events-api-v2/trigger-events/)

### イベントを変更する
インシデントの状態(Triggeredなど)をAPIを経由して変更することができる
変更した内容は各サービスの「recent changesで確認することができる」
[Send a Change Event](https://developer.pagerduty.com/docs/events-api-v2/send-change-events/)


# SDK
最初「APIでデータ取得とか操作するものなのか」と思っていたらSDKがあった
[API Client Libraries](https://developer.pagerduty.com/docs/tools-libraries/client-libraries/)



# 参考
[Incidents](https://support.pagerduty.com/docs/incidents)
[PagerDutyのエスカレーションポリシーとサービス](https://thinkit.co.jp/article/13420)
