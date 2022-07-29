# what
- fluentdについて、基本概要等を記載する
    - aggregaterについても記載
- 場合によっては環境立てて触ってみた、とかも書くかも


# [fluentd](https://www.fluentd.org/)とは
- ログ収集ソフトウェア
- ログの取り込み(インプット)、別システムへの出力(アウトプット)を制御する
- 収集したログはファイル, RDMS, NoSQL, Hadoop等に書き込むことが可能
- embulkと同様、pluginを利用して拡張することも可能

# 実際に触ってみる
- 以下ブログ記事を参考にfluentdを軽く動かしてみる
    - [fluentd の基礎知識](https://abicky.net/2017/10/23/110103/)
- ruby環境を利用してfluentdをインストールした

## 設定ファイルについて
- 今回作成したファイルは`fluent.conf`
- 「echoでfluentdにテキストを送り、それをログとして出力する」という内容

```conf
## built-in TCP input
## $ echo <json> | fluent-cat <tag>
<source>
  @type forward
  @id forward_input
</source>

## match tag=debug.** and dump to console
<match debug.**>
  @type stdout
  @id stdout_output
</match>
```

### タグ
- **source** ...input plugin の設定
- **match** ...ルールにマッチしたタグのイベントを処理する output plugin の設定
- **@type**...使用する plugin の名前
- **@id**...識別子で fluentd 自身のログ等に使われる

 タグについてはここらへん読むと良さそう
[List of Directives](https://docs.fluentd.org/configuration/config-file#list-of-directives)

### 各処理の意味
- `<source> @type forward`
    - 意味: type forwardでfluentdにtcpで入力される
- `<match debug.**>`
    - 意味: source type forwardで入力された`"debug.<文字列>"`のタグがついたものはfluentdの標準出力で表示する
