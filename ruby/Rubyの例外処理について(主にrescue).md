# what
rubyの例外処理で利用されるrescueについて、調べたことや理解したことを記載する
そもそも例外処理とは?というところについても理解する

# 例外とは?
例外: **プログラム実行中のエラー**のことを指す。
プログラミングに置いては様々な種類のエラーが存在する。例えば以下のようなエラー
- システムエラー
- サーバー落ち
- ヒューマンエラー
- 入力の失敗
- コードエラー
- ;がない

**コード内で起きたエラー=例外**と考えて良い。

Rubyで例外を補足するためのメソッドとして**resucue**が存在する。
resucueメソッドを利用すれば、例外が発生した時にどう対処するか予め設定しておくことが可能になる。

## なぜresucueで対応する必要があるのか?
しかし、なぜ例外が発生した時にresucueで対処する必要があるのか?

例外処理を書いておかないと、例外が発生した時点でプログラムが止まってしまい、**それ以降に記述してあるコードが読み込まれない**ため。
そうなった場合、もし処理に必要なコードが例外発生より後に書いてあった場合にはプログラムが破綻してしまう。
なので、例外が発生した際にどのような処理をするのかを設定する必要がある。

## 例外の種類
例外にもいろいろな種類が存在する。
- 0で乗算したときのZeroDivisionError
- メモリ確保に失敗した時に発生するNoMemoryError
- 定義されていないローカル変数や定数を使用した時に発生するNameError
等など

例外の種類はRubyの公式マニュアルにも記載されているので、詳しく知りたい場合はそれを参照
[Ruby 2.1.0 リファレンスマニュアル > ライブラリ一覧 > 組み込みライブラリ](http://doc.okkez.net/2.1.0/view/library/_builtin)

# 例外を処理するbegin~rescueメソッド
## 基本的なrescue
基本的なrescueメソッドの使い方を実際に触りながら理解していく

```ruby
begin
  1/0
rescue
  puts '0で割ってはいけません'
end
```

実行結果
```
$ bundle exec ruby basic-rescue.rb
0で割ってはいけません
```

このプログラムでは、「1/0」と言う箇所で整数は0で割れないので、「ZeroDivisionError」という例外が発生する。
beginブロックで例外が発生すると、beginブロックの処理は中断され、処理のフローはrescueに移る。
そして、ここではrescueブロックで"0で割ってはいけません"と表示している。
その後、beginブロックには復帰せず、endまで処理が進んでいる。

### 例外の補足
rescueでは、具体的にどんなエラーが起きたのか、その種類も補足することができる。

```ruby
begin
  1/0
  rescue => e
    puts e
end
```

実行結果
```
divided by 0
```

上記のコードを実行すると、変数eに格納された例外が何故起きたのか教えてくれる。
この場合はどの例外を処理するは省略されているが、厳密にはStandardErrorクラスのサブクラスに当たる例外だけを補足する。

ZeroDivisionErrorクラスはStandardErrorクラスを継承しているので、この場合は例外を補足することができる。

また、eには例外オブジェクトが代入されるので、それら例外オブジェクトに実装されている基本的なメソッドを使用することができる。

```ruby
begin
  1/0
rescue StandardError => e
  puts e
  puts e.class
  puts e.class.superclass
  puts e.message
end
```

実行結果
```
divided by 0
ZeroDivisionError
StandardError
divided by 0
```

## 例外の種類を分ける
エラーの種類に応じて例外処理を変えることも可能

```ruby
begin
  1/0
  rescue ZeroDivisionError => e
    puts e.class
    puts e.message
end
```

実行結果
```
ZeroDivisionError
divided by 0
```


