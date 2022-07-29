# [Redash](https://redash.io/product/)とは
- Arik Fraimovich氏が中心となって開発しているBIツール(OSS/Saas)
- TresueDataやBigQuery等の様々なデータソースに対応している
- クエリエディタやだけでなく、グラフ化・ダッシュボード等の機能を備えている
- ブラウザベースのツールなので、サーバー等の構築とアカウントを作成すればだれでも利用ができる
- OSS版を利用することによって会社や利用目的に合わせてカスタマイズできる
- サーバー管理などが不要なSaas版が有償で提供されている
    - [Pricing](https://redash.io/pricing/)

# Redashの基本構成について
会社のRedash構成を振り返る前に、Redashそのものの基本構成について大まかに記載する
今回記載する内容は公式で提供しているDocker版のRedashについて(社内Redashはこちらを利用している)
[redash/redash dockerhub](https://hub.docker.com/r/redash/redash)

- 参考
    - [一歩踏み込むRedash](https://speakerdeck.com/ariarijp/bu-ta-miip-mu-redash?slide=11)
        - 上記資料はUbunt版(現時点では廃止)だが、基本構成は一緒なので掲載
    - [Redashについていろいろ調べるときに頭のなかでイメージしている図](http://ariarijp.hatenablog.com/entry/2018/01/03/224031)

**基礎構成**
![redash構成](./redash%E6%A7%8B%E6%88%90%E5%9B%B3_%E5%9F%BA%E6%9C%AC.png)

### 各構成要素について
- nginx: webサーバのリバースプロキシの役割をになっている
    - ref: [リバースプロキシ とは](https://wa3.i-3-i.info/word1755.html)
    - サーバーの代わりになって外部からのアクセスを受け付けたり、そのサーバーで提供しているコンテンツを外部のユーザーに提供するプロキシのこと
- redash/python: Redash本体の部分
    - redash server: Redashのweb画面での表示部分を担当している。Flaskで動いている。
    - redash worker: クエリを実行している部分。データソースと接続しているのは主にここ。Celeryを使って動いている。スケジュール実行もここ。
- PostgreSQL: Redashにおける管理情報(アカウント, クエリ実行記録等)をまとめている部分
- Redis: 複数のクエリの実行リクエストが来た場合、実行待ちのクエリを一時的に格納しておく部分
- データソース: BigQueryやRDSといった連携先のDB
