# what

- embulkについて調べてわかったことや実際に試してみたことについて記載する

# [embulk](https://www.embulk.org/)について
- > Embulkは、さまざまなストレージ、データベース、NoSQL、クラウドサービス間のデータ転送を支援する並列バルクデータローダーです。(公式文翻訳より)
- **ファイルやデータベースからデータの抽出を行い、別のストレージやデータベースに転送**するためのツール(OSS)
- 公式サイトはあるが、GitHubにも記載がある
    - [embulk  --GitHub](https://github.com/embulk/embulk)
- 基本的な概要は開発者の古橋さんが書いた記事もあったりする
    - [並列データ転送ツール『Embulk』リリース！](http://frsyuki.hatenablog.com/entry/2015/02/16/080150)
- 入力、出力、データ加工をプラグインを書くことにより拡張(カスタマイズ)して利用することができるのが大きな特徴


# 実際に触ってみる
- 公式チュートリアル(クイックスタート)があるのでそれを試してみる
    - [Quick Start](https://github.com/embulk/embulk#quick-start)
- コマンドに関しては下記にまとめている方もいた
    - [Embulk example](http://www.ne.jp/asahi/hishidama/home/tech/embulk/example.html#example.yml)

### 作業環境
- embulk... embulk 0.9.18
- java8

## JREインストール&path通す
- emblukは以前インストールしていた(brew)
- 自分はzshを利用しているのでpathの指定方法が少々異なる

```zsh
$ curl --create-dirs -o ~/.embulk/bin/embulk -L "https://dl.embulk.org/embulk-latest.jar"
$ chmod +x ~/.embulk/bin/embulk
$ echo 'export PATH="$HOME/.embulk/bin:$PATH"' >> ~/.zshrc
$ source ~/.zshrc
```

## サンプル用意
- `example`コマンド出サンプルデータを生成する

```zsh
# サンプルデータを生成
$ embulk ./try1


# サンプルデータから設定ファイル(config.yml)を生成
$ embulk guess ./try1/seed.yml -o cofig.yml

# 入力データの読み込み
$ embulk preview config.yml

# 転送する
$ embulk run config.yml
```

## プラグインをインストール
- `embulk gem install <name>`でプラグインをインストールできる

```zsh
$ embulk gem install embulk-output-command

# インストールされているプラグイン一覧を確認する
$ embulk gem list
2019-11-29 13:29:33.309 +0900: Embulk v0.9.20


*** LOCAL GEMS ***

bundler (1.16.0)
did_you_mean (default: 1.0.1)
embulk (0.9.20 java)
embulk-output-command (0.1.4)
jar-dependencies (default: 0.3.10)
jruby-openssl (0.9.21 java)
jruby-readline (1.2.0 java)
json (1.8.3 java)
liquid (4.0.0)
minitest (default: 5.4.1)
msgpack (1.1.0 java)
net-telnet (default: 0.1.1)
power_assert (default: 0.2.3)
psych (2.2.4 java)
rake (default: 10.4.2)
rdoc (default: 4.2.0)
test-unit (default: 3.1.1)
```




## 設定ファイルについて
- 設定ファイルとは、今回においては`config.yml`のこと

```yml
in:
  type: file
  path_prefix: /Users/<user名>/tutorial_embulk/./try1/csv/sample_
  decoders:
  - {type: gzip}
  parser:
    charset: UTF-8
    newline: LF
    type: csv
    delimiter: ','
    quote: '"'
    escape: '"'
    null_string: 'NULL'
    trim_if_not_quoted: false
    skip_header_lines: 1
    allow_extra_columns: false
    allow_optional_columns: false
    columns:
    - {name: id, type: long}
    - {name: account, type: long}
    - {name: time, type: timestamp, format: '%Y-%m-%d %H:%M:%S'}
    - {name: purchase, type: timestamp, format: '%Y%m%d'}
    - {name: comment, type: string}
out: {type: stdout}
```

- 意味合いとしては下記の様になる様子
    - in: インプットプラグインに関する記述
    - out: アウトプットプラグインに関する記述
    - filters: フィルタプラグイン関する記述
    - exec: エクゼキュータに関する記述
