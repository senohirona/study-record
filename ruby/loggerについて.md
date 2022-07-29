# what
rubyのloggerについて調べたり実際に触って理解したこと等を記載する

# logレベルとは
logにはlogレベルと呼ばれる概念が存在する。
logレベルによってlogの重要度が変わっていき、logレベルによって出力するログを分けたりできる。

logレベルは以下の６種類に分けることができる
上に行くほどログレベルが高いと呼ばれ、より重要な情報を表しているログを表す。
|logレベル|意味|
| --- | --- |
| UNKNOWN |常にログとして出力される必要がある未知のメッセージ  |
| FATAL | プログラムをクラッシュさせるような制御不可能なエラー |
| ERROR | エラー |
| WARN |  警告|
| INFO | 一般的な情報 |
| DEBUG | 低レベルの情報 |

# loggerとは
loggerとは、Rubyからlogを出力するユーティリティーのこと。
log出力に関する様々な便利な機能を搭載している。

logは例えば以下のような出力をする

```
# Logfile created on 2017-09-21 11:15:52 +0900 by logger.rb/56815
W, [2017-09-21T11:15:52.517941 #8136]  WARN -- : ログ3
W, [2017-09-21T11:16:49.831474 #26884]  WARN -- : ログ3
```
以上のように
- ログを出力した日付
- ログを出力したRubyのプロセス番号(ここでは#8136や#26884)
- ログレベル(ここでは WARN : 警告)
- ログメッセージ(ここでは「ログ３」)

などが出力される。
ログはカスタマイズすることができ、エラーの発生シアファイル名や行番号を付与することもできる。


## 触ってみる
以下のようなコードを書いてみる

```ruby
# loggerライブラリを読み込む
require 'logger'

# ログオブジェクトを生成
log = Logger.new('./rubyfile.log')

# 各ログレベルのログメッセージを'/tmp/rubyfile.log'に出力
log.debug('*debug log')
log.info('*info log')
log.warn('*warn log')
log.error('*error log')
log.fatal('*fatal log')
log.unknown('*unknown log')
```

実行してみる

```rubyfile.log
# Logfile created on 2021-03-16 13:31:10 +0900 by logger.rb/v1.4.2
D, [2021-03-16T13:31:10.002570 #88750] DEBUG -- : *debug log
I, [2021-03-16T13:31:10.002617 #88750]  INFO -- : *info log
W, [2021-03-16T13:31:10.002643 #88750]  WARN -- : *warn log
E, [2021-03-16T13:31:10.002663 #88750] ERROR -- : *error log
F, [2021-03-16T13:31:10.002683 #88750] FATAL -- : *fatal log
A, [2021-03-16T13:31:10.002701 #88750]   ANY -- : *unknown log
```

Logger.newでloggerオブジェクトを作成した後は、loggerのdebug, info, warn, error, fatal, unknownメソッドを使ってログレベル毎に適したフォーマットでログが出力できる。


# loggreの基本的な使い方
## 基本的なlogの出力方法
loggerの基本的な使い方を見ていく
以下のコードでは、テキストファイルを読み込み、テキストファイルの各行を１行ずつ読み込んでいる。
各業の行頭が`#`で始まっていない場合は、エラーとみなし、loggerオブジェクトを使ってログに出力している。
エラーでなければ、putsで標準出力に行を表示している。

```ruby
# loggerライブラリを読み込む
require 'logger'

# loggerを生成し、最大ログ出力レベルをLogger::DEBUGに限定
logger = Logger.new('./filescan.log')
logger.level = Logger::DEBUG

# データファイル
filename = "sample.txt"

begin
  # データファイル"sample.txt"を行ごとに読み込む（スキャンする）
  File.foreach(filename) do |line|
    # 行頭が#で始まっていなければエラー
    unless line.start_with?("#")
      # エラーをログする
      logger.error("スキャンエラー: #{line.chomp}")
    else
      # エラーでなければ、標準出力に出力する
      puts line
    end
  end
rescue => err
    # 例外が発生した場合は、その旨をログする
    logger.fatal("例外発生:#{err.message}")
end
```

実行結果

```filescan.log

# Logfile created on 2021-03-16 14:33:44 +0900 by logger.rb/v1.4.2
F, [2021-03-16T14:34:12.683924 #89445] FATAL -- : 例外発生:No such file or directory @ rb_sysopen - sample.txt
```


## ログの出力先指定
ログの出力先を指定することができる。

出力先は主にファイルだが、標準出力(STDOUT)や標準エラー出力(STDERR)等も指定することができる。

```ruby
# loggerライブラリを読み込む
require 'logger'

# 標準出力へのログ
logger1 = Logger.new(STDOUT)
#標準エラー出力へのログ
logger2 = Logger.new(STDERR)
#ファイルへのログ
logger3 = Logger.new('tofile1.log')

#ファイルへのログ指定方法その２
file = File.open('tofile2.log', File::WRONLY | File::APPEND | File::CREAT)
logger4 = Logger.new(file)

# 各種loggerへ書き込み
logger1.error("ログ1")
logger2.info("ログ2")
logger3.warn("ログ3")
logger4.debug("ログ4")
```

実行結果(標準出力, 標準エラー出力)
```
$ bundle exec ruby test-logger.rb
E, [2021-03-16T15:21:48.975564 #89962] ERROR -- : ログ1
I, [2021-03-16T15:21:48.975618 #89962]  INFO -- : ログ2
```

実行結果(tofile1.log)
```
# Logfile created on 2021-03-16 15:21:38 +0900 by logger.rb/v1.4.2
W, [2021-03-16T15:21:48.975629 #89962]  WARN -- : ログ3
```

実行結果(tofile2.log)
```
D, [2021-03-16T15:21:48.975660 #89962] DEBUG -- : ログ4
```

標準出力と標準エラー出力にもログを出力できている。

### メモ
- 標準出力にログを出力する場合は、Logger.newメソッドに「STDOUT」を指定する。
- 標準エラー出力にログを出力する場合は、Logger.newメソッドに「STDERR」を指定する。
- ファイルにログを出力する場合は、Logger.newメソッドにファイル名を指定する。
    - File.openメソッドで開いたファイルにログを出力する場合は、Logger.newメソッドにそのファイルのオブジェクトを指定する。

## ログレベルの指定方法
ログレベルの指定はlogggerオブジェクトのerror, info, warn, debug等のメソッドを使用することで、ログレベルをそれぞれERROR, INFO, WARN, DEBUG等のしたことになる。

また、ログに出力する最大ログレベルを指定することができる。

これの最大ログレベルをプログラムで切り替えられるようにすることで、本番系のシステムとテスト系のシステムで、ログの出力情報の量や質を変更できる。

ログ最大レベルはLoggerのlevelプロパティにLoggerの定数(Logger::WARN等)を指定することで有効になる

```ruby
# loggerライブラリを読み込む
require 'logger'

logger = Logger.new('loglevel.log')

# ログ最大出力レベルをWARNに
# ログ優先度レベルはDEBUG<INFO<WARN<ERROR<FATAL<UNKNOWN
logger.level = Logger::WARN

# ログ出力
logger.error("ログ1")
logger.info("ログ2")
logger.warn("ログ3")
logger.debug("ログ4")
```

実行結果(loglevel.log)
```
# Logfile created on 2021-03-16 18:26:18 +0900 by logger.rb/v1.4.2
E, [2021-03-16T18:26:18.209270 #91886] ERROR -- : ログ1
W, [2021-03-16T18:26:18.209316 #91886]  WARN -- : ログ3
```

以上のようにlevelに`Logger::WARN`を指定した。

ログの優先度レベルはDEBUG<INFO<WARN<ERROR<FATAL<UNKNOWNの順になるため、上記のコードではDEBUGとINFOのログの情報は出力されない。
また、WARN以上のログレベルであるERRORとWARNのリグの情報は出力されている。

## スタックトレースに改行を付けてログ出力
ログはデバッグする際に便利な情報。
ここで、エラーや例外が発生した箇所のスタックトレースをログに出力できれば便利。


スタックトレース: プログラムを実行しているときの現在地のメソッド名やローカル変数の情報をメソッド呼び出しのたびに積み上げて(スタック)保存している情報のこと。
その情報をデバッグ時にプログラムのどのメソッドやローカル変数の情報を得ているか追跡(トレース)しやすいように加工したものを総称してスタックトレースという。

通常、例外のスタックトレースは配列になっている。
そのため、配列のjsonメソッドに "\n"をしていて、配列から"\n"で区切られた文字列に変換する。

例外オブジェクトをerrとすると、スタックトレースは「err.backtrace.join("\n")」で、スタックを1行ごとに開業したテキストになる。
この情報をログに出力してみる。

```ruby
# loggerライブラリを読み込む
require 'logger'

# loggerを生成し、最大ログ出力レベルをLogger::DEBUGに限定
logger = Logger.new('./filescan2.log')
logger.level = Logger::DEBUG

# データファイル"sample.txt"
filename = "sample.txt"

begin
  # データファイル"sample.txt"を行ごとに読み込む
  File.foreach(filename) do |line|
    #行頭が#で始まっていなければ、例外
    unless line.start_with?("#")
      #例外を投げる
      throw
    else
      # エラーでなければ、標準出力に出力する
      puts line
    end
  end
  rescue => err
    # スタックトレースに改行を付けてログ出力する
    logger.fatal(err.backtrace.join("\n"))
end
```

実行結果(./filescan2.log)
```
# Logfile created on 2021-03-16 18:46:06 +0900 by logger.rb/v1.4.2
F, [2021-03-16T18:46:16.702142 #92042] FATAL -- : test-logger.rb:13:in `foreach'
test-logger.rb:13:in `<main>'
```

ファイル「sample.txt」を1行ずつ読み込み、各行が"#"で始まっていなければエラーとして例外を投げている。

例外が投げられるとresucueブロッグに処理が移る、
resucueブロックにて、例外に含まれるスタックトレースの情報をログに出力している、

実行結果を見てみると、スタックトレースの情報をスタックあたり1行ずつ出力していることがわかる。
これで、エラーの発生箇所がわかりやすくなる。


# 参考
[【Ruby入門】loggerの使い方まとめ](https://www.sejuku.net/blog/19387)
