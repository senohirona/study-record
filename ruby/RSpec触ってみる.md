# what
PagerDutyのincident定期通知botタスクで作成したスクリプトをRSpecを利用してテストしてみる、という物があるので、やってみる
そもそもRSpecについてわからない部分もあるのでそれについて調べたことについても記載する。

# RSpecとは
RubyやRailsで作ったクラスやメソッドをテストするためのフレームワーク
[公式サイト](https://rspec.info/)
[github](https://github.com/rspec/rspec-rails)

動く仕様書(Spec) として自動テストを書くという発送で作られており、要求仕様をドキュメントに記述するような間隔でテストケースを記述することができる。
rubyよりRailsで利用されることが多い。


# 導入
以下の動画と記事を参考にRSpecを挙動確認用のディレクトリをつくって環境構築してみる
- [【2019年版】RailsじゃないRspec3環境を構築する方法](https://www.youtube.com/watch?v=njFJ4FOOKC4&feature=youtu.be)
- [RailsじゃないRspec3環境を構築する方法](https://qiita.com/yusabana/items/db44b81bdddf6ed0e9f5)
<こちらの環境の各version>
- ruby: 2.6.5
- RSpec: 3.10.0

# 基礎概念
RSpecを利用するには`テストコード`と`プロダクション(プロダクト)コード`の2つが最低限存在する
プロダクションコードは「テスト対象となるコード」のことで、自分や他のエンジニアが機能開発等で書いたコードを指す。
テストコードは「プロダクションコードをテストするコード」のことで、テストコードに書かれた内容をもとにプロダクションコードをテストする。

テストコードは原則`spec`ディレクトリ配下に置かれる(例: `spec/**/**_spec.rb`)

# 触ってみる(RSpec入門その1)
以下の記事を参考にRSpecの書き方がどういったものなのかを実際に触ってみる。
[使えるRSpec入門・その1「RSpecの基本的な構文や便利な機能を理解する」](https://qiita.com/jnchito/items/42193d066bd61c740612)

今回はプロダクションコードはlibディレクトリ配下に、テストコードをspecディレクトリ配下に置いた。
↓初期配置
```
practice_rspec
├── Gemfile
├── Gemfile.lock
├── .rspec
├── lib
│   └── hello.rb
└── spec
    ├── hello_spec.rb
    └── spec_helper.rb
```


## describe / it / expect

```test.rb
class Test
  def plus
    1+1
  end
end
```

```test_spec.rb
require_relative '../lib/test'
RSpec.describe Test do
  it "1+1=2" do
    expect(Test.new.plus).to eq 2
  end
end
```

`describe(RSpec.describe)`はテストのグループ化を宣言する
(今回はTestという名前のテストを書く、と宣言している)

`it`はテストをexampleという単位にまとめる役割をする。
`it do ... end`の中のエクスペクテーション(期待値と実際の値の比較)がすべてパスすれば、そのexampleはパスしたことになる。

`expect(X).to eq Y`で記述するのがエクスペクテーション。
expectには「期待する」という意味があるので、`expect(X).to eq Y`は「XがYに等しくなることを期待する」と読める。
今回のテスト`expect(Test.new.plus).to eq 2`は「TestクラスのPlusメソッドの値が2になる事を期待する」というテストになる。

### 複数のexample
`describe`の中には複数のexample(it do ... end)を書くことができる

```test.rb
class Test
  def plus
    1+1
  end
  def minus
    10-1
  end
end
```

```test_spec.rb
require_relative '../lib/test'
# method別にテストを書いている
RSpec.describe Test do
  it "1+1=2" do
    expect(Test.new.plus).to eq 2
  end
  it "10-1=9" do
    expect(Test.new.minus).to eq 9
  end
end
```

原則は「１つのexampleにつき１つのエクスペクテーション」で書いたほうがテストの保守性が良くなる。

## context/before
RSpecには`describe`以外にも`context`という機能でテストをグループ化することもできる。
どちらも機能的には同じだが、`context`は条件を分けたりするときに利用する場合が多い。

```user.rb
class User
  def initialize(name:, age:)
    @name = name
    @age = age
  end
  def greet
    if @age <= 12
      "ぼくは#{@name}だよ"
    else
      "僕は#{@name}です"
    end
  end
end
```

```user_spec.rb
require_relative '../lib/user'

RSpec.describe User do
  describe '#greet' do
    context '12歳以下の場合' do
      it 'ひらがなで答えること' do
        user = User.new(name: 'たろう', age: 12)
        expect(user.greet).to eq 'ぼくはたろうだよ'
      end
    end
    context '13歳以上の場合' do
      it '13歳以上の場合、漢字で答えること' do
        user = User.new(name: 'たろう', age: 13)
        expect(user.greet).to eq '僕はたろうです'
      end
    end
  end
end
```


`before do ...end`で囲まれた部分はexampleの実行前に毎回呼ばれる。
`before`ブロックの中では、テストを実行する前の共通処理やデータのセットアップ等を行うことが多い。
今回のサンプルコードでは`name: 'たろう'`が重複しているのでDRY(繰り返しや重複を避ける)にしてみる。

```user_spec.rb
require_relative '../lib/user'

RSpec.describe User do
  describe '#greet' do
    before do
      @params = {name: 'たろう'}
    end
    context '12歳以下の場合' do
      it 'ひらがなで答えること' do
        user = User.new(@params.merge(age: 12))
        expect(user.greet).to eq 'ぼくはたろうだよ'
      end
    end
    context '13歳以上の場合' do
      it '漢字で答えること' do
        user = User.new(@params.merge(age: 13))
        expect(user.greet).to eq '僕はたろうです'
      end
    end
  end
end
```

## ネストしたdescribeやcontextの中でbeforeを使う
`before`は`describe`や`context`毎に用意することができる。
`escribe`や`context` がネストしている場合は、親子関係に応じて`before`が順番に呼ばれる。

```user_spec.rb
require_relative '../lib/user'

RSpec.describe User do
  describe '#greet' do
    before do
      @params = {name: 'たろう'}
    end
    context '12歳以下の場合' do
      before do
        @params.merge!(age: 12)
      end
      it 'ひらがなで答えること' do
        user = User.new(@params)
        expect(user.greet).to eq 'ぼくはたろうだよ'
      end
    end
    context '13歳以上の場合' do
      before do
        @params.merge!(age: 13)
      end
      it '漢字で答えること' do
        user = User.new(@params)
        expect(user.greet).to eq '僕はたろうです'
      end
    end
  end
end
```

今回の変更の場合、`@params = {name: 'たろう'}`の部分は、「12歳以下の場合」でも「13歳以下の場合」でも呼ばれる。
一方、`params.merge!(age: 12)`と`@params.merge!(age:13)`はそれぞれ「12歳以下の場合」と「13歳以上の場合」の中でしか呼ばれない。
呼ばれる順番は親、子の順。

## 応用: インスタンス変数の代わりにletを使う
先程のコードの例では`@params`を利用していた。
RSpecdではこれを`let`に置き換えることができる。


```user_spec.rb
require_relative '../lib/user'

RSpec.describe User do
  describe '#greet' do
    let(:params) {{name: 'たろう'}}
    context '12歳以下の場合' do
      before do
        params.merge!(age: 12)
      end
      it 'ひらがなで答えること' do
        user = User.new(params)
        expect(user.greet).to eq 'ぼくはたろうだよ'
      end
    end
    context '13歳以上の場合' do
      before do
        params.merge!(age: 13)
      end
      it '漢字で答えること' do
        user = User.new(params)
        expect(user.greet).to eq '僕はたろうです'
      end
    end
  end
end
```

`let(:foo) {...}`のように書くと、`{...}`の中の値が`foo`として参照できる、というのが`let`の基本的な使い方。
ただし、上の例では` {{name: 'たろう'}}` と`{}`が2回出てくるのでややこしくなっている。
外側の`{}`はRubyのブロックで、内側の`{}`はハッシュリテラル。


## 応用: userをletにする
インスタンス変数だけではなく、ローカル変数を`let`で置き換える事もできる

```user_spec.rb
require_relative '../lib/user'

RSpec.describe User do
  describe '#greet' do
    let(:user) {User.new(params)}
    let(:params) {{name: 'たろう'}}
    context '12歳以下の場合' do
      before do
        params.merge!(age: 12)
      end
      it 'ひらがなで答えること' do
      　# ここで定義していたuser=User.newが無くなる
        expect(user.greet).to eq 'ぼくはたろうだよ'
      end
    end
    context '13歳以上の場合' do
      before do
        params.merge!(age: 13)
      end
      it '漢字で答えること' do
       # ここで定義していたuser=User.newが無くなる
        expect(user.greet).to eq '僕はたろうです'
      end
    end
  end
end
```

重複していた`user=User.new(params)`の部分を共通化することができた。

## 応用: ageをletで置き換える
`params.merge!(age: 12)`の部分も`let`を利用して書くことができる。

```user_spec.rb
require_relative '../lib/user'

RSpec.describe User do
  describe '#greet' do
    let(:user) {User.new(params)}
    let(:params) {{name: 'たろう', age: age}}
    context '12歳以下の場合' do
      let(:age) {12}
      it 'ひらがなで答えること' do
        expect(user.greet).to eq 'ぼくはたろうだよ'
      end
    end
    context '13歳以上の場合' do
      let(:age) {13}
      it '漢字で答えること' do
        expect(user.greet).to eq '僕はたろうです'
      end
    end
  end
end
```

**letのメリット**
`let`は「`before`+インスタンス変数」を使うときとは異なり遅延評価される、という特徴がある。
遅延評価: とあるプログラムに置いて変数や引数の値が必要になるまで評価されない、というもの。引数や変数が参照されるときに評されるということになるので、引数が変数が必要となるまでは結果として引数や変数に渡すメソッドを実行することがないので、プログラムの高速化が期待できる。
参考: [Rubyで遅延評価を実行する方法を現役エンジニアが解説【初心者向け】](https://techacademy.jp/magazine/19922)

letは必要になる瞬間まで呼び出されない。

上のコードを例にすると
1. expect(user.greet).to が呼ばれる->userとは?
1. let(:user){User.new(params)}が呼ばれる->paramsとは?
1. let(:params){{name:'たろう', age: age}}が呼ばれる->ageとは?
1. let(:age){12} (または13)がyボアれる
1. 結果としてexpect(User.new(name: 'たろう',  age: 12).greet).to を呼んだことになる


## テスト対象のオブジェクトを1箇所にまとめる
テスト対象のオブジェクト(もしくはメソッドの実行結果)が明確に1つに決まっている場合は、`subject`を利用してテストコードをDRYすることができる。

今回のコードデーはどちらのexampleも`user.greet`の実行結果をテストしているので、user.greetをsubjectに引き上げて、exampleの中から消してみる。

```user_spec.rb
require_relative '../lib/user'

RSpec.describe User do
  describe '#greet' do
    let(:user) {User.new(params)}
    let(:params) {{name: 'たろう', age: age}}
    subject{user.greet}
    context '12歳以下の場合' do
      let(:age) {12}
      it 'ひらがなで答えること' do
        is_expected.to eq 'ぼくはたろうだよ'
      end
    end
    context '13歳以上の場合' do
      let(:age) {13}
      it '漢字で答えること' do
        is_expected.to eq '僕はたろうです'
      end
    end
  end
end
```

さらに、`it`にわたす文字列を省略してみる

```user_spec.rb
require_relative '../lib/user'

RSpec.describe User do
  describe '#greet' do
    let(:user) {User.new(params)}
    let(:params) {{name: 'たろう', age: age}}
    subject{user.greet}
    context '12歳以下の場合' do
      let(:age) {12}
      it {is_expected.to eq 'ぼくはたろうだよ'}
    end
    context '13歳以上の場合' do
      let(:age) {13}
      it {is_expected.to eq '僕はたろうです'}
    end
  end
end
```

これでテストコードは完成

## 注意
無理にletやsubjectを多用する必要はない。
逆にコードが読みにくくなる可能性があるため。

# RSpec入門その2
以下の記事を参考に入門その1より少し発展した内容を知る
[使えるRSpec入門・その2「使用頻度の高いマッチャを使いこなす」](https://qiita.com/jnchito/items/2e79a1abe7cd8214caa5)

## マッチャとは
マッチャ(matcher)は「期待値と実際の値を比較して、一致した(もしくは一致しなかった)という結果を返すおオブジェクト」のこと。

```ruby
expect(1+2).to eq 3
expect([1,2,3]).to include 2
```

上記のサンプルコードで言うと`eq`や`include`がマッチャに当たる。
マッチャは自分自身に定義されている検証ルールに従って、実際の値(1+2や[1,2,3])と期待値(3や2)を比較し、ルールに合致しているかどうかを判断する。

## to/not_to/to_not
「~であること」を期待する場合は`to`を使う
```ruby
expect(1+2).to eq 3
```
反対に「~ではないこと」を期待する場合は`not_to`もしくは`to_not`を使う
(**ちなみに、not_toもto_notも同じ意味なので、どちらを使っても問題はない**)
```
expect(1+2).not_to eq 1
expect(1+2).to_not eq 1
```

ちなみに、`to`や`not_to`自体はマッチャではない。
マッチャの実行結果を受け取って、テストをパスさせるか否かを判断するRSpecのメソッド

## eq
期待値と実際の値が「等しい」かどうかを検証する場合は`eq` を使う。
```ruby
expect(1+2).to eq 3
```

## be
`be`は等号・不等号と組み合わせて、値の大小を検証するときによく使われるマッチャ。
以下のコードの場合は「1+2が3以上であること」を検証している。

```ruby
expect(1+2).to be >= 3
```


### beとeqの違い
どちらも似たような用途で使うこともできるが、微妙に動きが異なるので注意が必要。
(ややこしいので最初はサラッと見るのが良さそう)

`be`は等号・不等号なしで次のように書くとも出来る
```ruby
message = 'Hello'
expect([message].first).to be message
```
等号・不等号なしで書いた場合は、2つの値が同一のインスタンスかどうかを検証する。
つまり、`equal?`メソッドの結果がtrueかどうかを検証していることになる。

同じ値でもインスタンスが異なる場合はテストは失敗する

```ruby
message_1 = 'Hello'
message_2 = 'Hello'
# message_1とmessage_2は異なるインスタンスなのでテストが失敗する
expect([message_1].first).to be message_2
```

一方、beの代わりにeqを使うと、`==`を使って比較するので、先程のテストはパスする
```ruby
message_1 = 'Hello'
message_2 = 'Hello'
# message_1 == message_2 の結果は真になるのでテストはパスする
expect([message_1].first).to eq message_2
```

同一インスタンスかどうかを検証する機会はあまり多くないらしいので、大体のテストケースでは`eq`を利用するとOKであろう。

<まとめ>
- expect(A).to be B は A.equal?(B) が真であれば（つまり、同一インスタンスであれば）パスする
- expect(A).to eq B は A == B が真であれば（つまり、同値であれば）パスする
- true やシンボルのように、「同じ値は常に同じインスタンス」になる場合は eq と be を入れ替えても同じ結果になる

## be_xxx(predicateマッチャ)
RSpecでは、`empty?`のようにメソッド名が「?」で終わり、戻り値がtrue/falseになるメソッドを`be_empty`のような形式で検証できるのが特徴。

```ruby
expect([]).to be_empty
```
上記のコードは以下のコードと同じ意味になる
```ruby
expect([].empty?).to be true
# または
expect([].empty?).to eq true
```

`be_xxx`のようなpredicate(述語)マッチャを使う利点は、テストコードが自然な英文のように読める点。
("expect user to be valid" のように読める)

## be_truthy / be_falsey
「?」で終わらないが、戻り地としてtrue/falseを返すメソッドは`be_truthy` / `be_falsey`というマッチャで検証することができる。

例えば、Railsで`save`メソッドを呼ぶと、保存に成功した場合は`true`、失敗した場合は`false`が帰ってくる。
なので、`be_truthy` / `be_falsey`を使って以下のようにテストを書くことができる

```sample.rb
class User < ActiveRecord::Base
  # nameとemail情報の入力が必須になっている。
  validates :name, :email, presence: true
end
```

```sample_spec.rb
# 必須項目が入力されていないので保存できない(結果はfalse)
user = User.new
expect(user.save).to be_falsey

# 必須項目が入力されているので保存できる(結果はtrue)
user.name = 'Tom'
user.email = 'tom@example.com'
expect(user.save).to be_truthy
```

### be_truthy/be_falseyとbe true/be false との違い
Rubyでは「falseまたはnilであれば偽、それ以外はすべて真」と評価する。

be_truthy/be_falseyを使うと、その使用に合わせて戻り値の真偽を検証してくれる。
つまり「trueっぽい値」または「falseっぽい値」かどうかを検証してくれるのが`be_truthy`/`be_falsey`
一方、`be true`/`be false`を使うと、trueもしくはfalseで有ることを厳密に検証するので、それ以外の値を渡されるとテストで失敗する。

```ruby
# どちらもパスする
expect(1).to be_truthy
expect(nil).to be_falsey

# どちらも失敗する
expect(1).to be true
expect(nil).to be false

# be の代わりにeqを使った場合も同様に失敗する
expect(1).to eq true
expect(nil).to eq false
```

RSpecで真偽値を確認するテストを書く場合は、なにか特別な事情がない限り`be_truthy`/`be_falsey`を使ったほうが「Rubyらしい真偽値のテスト」ができる。


## change + from /to/ by
例えば以下のようなテストがあったとする

```ruby
# popメソッドを呼ぶと配列の要素が幻想することをテストする
x = [1,2,3]
expect(x.size).to eq 3
x.pop
expect(x.size).to eq 2
```
上記のテストは`change`を利用すると次のように書き換えることができる

```ruby
x = [1,2,3]
expect{x.pop}.to change{x.size}.from(3).to(2)
```

注意するのは`expect{x.pop}.to`のように、丸括弧ではなく中括弧を使っている点。
これはRubyの文法的にはブロックをexpectに渡している。
同様に、change{x.size}の部分でも中括弧を使っているので、ここもブロックを渡している。

`charge`マッチャを使ったテストコードは次のように読む。
`expect{X}.to change{Y}.from(A).to(B)` =「`XするとYがAからBに変わる事を期待する`」

changeのバリエーションとして、byを使った書き方もある。
byを使うと、「(もとの個数はともかく)1個減ること」を検証できる。

```ruby
x = [1,2,3]
expect{x.pop}.to change{x.size}.by(-1)
```
もちろん、増える場合も検証できる
```ruby
x = [1,2,3]
expect{x.pop}.to change{x.size}.by(1)
```

### changeを使った応用例
changeを使う場合は、もう少し複雑な処理を検証することが多い。
例としてRailsで「userを削除すると、userが書いたblogも削除されること」を検証してみる。
プロダクションコード↓
```ruby
class User < ActiveRecode::Base
# dependent: : destroyをつけたので、userを削除するとblogも削除される
  has_many :blogs, dependent: :destroy
end
```
テストコード↓
```ruby
it 'userを削除すると、userが書いたblogも削除されること' do
  user = User.create(name: 'Tom', email: 'tom@example.com')
  # userがblogを書いたことにする
  user.blogs.create(title: 'RSpec必勝法', content: 'あとで書く)'

  expect{user.destroy}.to change{Blog.count}.by(-1)
end
```

changeマッチャを使うと「Xに対するある操作が、一見無関係なYに影響を与える」といった検証内容を表現することができる。


## 時々使うマッチャ

### 配列+include
includeマッチャを使うと、「配列に~が含まれていること」を検証することができる。

```ruby
x = [1,2,3]
# 配列xに1が含まれていることを検証する
expect(x).to include 1
# 配列xに1と3が含まれていることを検証する
expect(x).to include 1,3
```

includeはhashや文字列に対しても使うことができる。
また`contain_exactly`というマッチャを使うと、配列の順番は無視して要素の個数や内容を検証することができる。

### raize_error
「エラーが起きること」を検証する場合は、`raise_error`マッチャを使うことができる。
例えば、「0で除算するとエラーが起きること」は次のようにテストできる。

```ruby
expect{1/0}.to raise_error ZeroDicisionError
```

もう少し実践的なサンプルコードを見ていく。
ここでは、自作クラスで明示的にエラーを返すようにしたメソッドをテストしている。


```raise_error.rb
class ShoppingCart
  def initialize
    @items = []
  end
  def add(item)
  # raiseは意図的に例外処理を発生させる際に利用する（今回はifの条件がtrueの場合、意図的に例外処理を発生させている)
    raise 'Item is nil.' if item.nil?
    @items << item
  end
end
```

```raise_error_spec.rb
it 'nilを追加するとエラーが発生すること' do
  cart = ShoppingCart.new
  expect{ cart.add nil }.to raise_error 'Item is nil.'
end
```

今回の例では`raise_error`の引数としてエラーメッセージ(文字列)を渡している。


# RSpec入門その3
モックを使ったテストについて学んでいく。

## モックとは
モックとは「本物のフリをする偽物のプログラム」のこと
何らかの理由で本物のプログラムが使えないもしくは使わないほうが良いケースの場合に利用される。
例) 外部のAPIを利用しなければならない場合


## 単純なモックの使い方
かんたんなサンプルプログラムを使ってみる

以下は天気予報をツイートする架空のTwitterbotのプログラム

```ruby
require 'twitter'

class WeatherBot
  def tweet_forecast
    twitter_client.update '今日は晴れです'
  end

  def twitter_client
    Twitter::REST::Client.new
  end
end
```

何も考えずにテストを書くとこうなる

```Ruby
it 'エラーなく予報をツイートすること' do
  weather_bot = WeatherBot.new
  expect{weather_bot.tweet_forecast}.not_to raise_error
end
```

しかし、このままでは全世界に向けてツイートを発信してしまうことになる。
実際にツイートしてしまうと、テスト中のメッセージが公開されてしまうだけでなく、他にも様々な問題を引き起こす。
例)短時間に何度も実行するとRate limit errorが起きる etc...

なのでTwitterと連携する部分はモックに置き換えて、テスト中にTwitterとの通信が発生しないように書き換える。

```ruby
it 'エラーなく予報をツイートすること' do
  # twitter clientのモックを作る
  twitter_client_mock = double('Twitter client')
  # updateメソッドが呼び出せるようになる
  allow(twitter_client_mock).to receive(:update)

  weather_bot = WeatherBot.new
  # twitter_clientメソッドが呼ばれたら上で作ったモックを返すように実装を書き換える
  allow(weather_bot).to receive(:twitter_client).and_return(twitter_client_mock)
  expect{weather_bot.tweet_forecast}.not_to raise_error
end
```

## 1. 空のモックオブジェクトを作る
最初に` Twitter::REST::Client`のニセモノ(モック)を作る

```
twitter_client_mock = double('Twitter client')
```

`double`というメソッドを使うと、モックオブジェクトを作れる。
(引数で渡す文字列は任意。省略することも可能)

テスト失敗時のメッセージ例
```
RSpec::Mocks::MockExpectationError: Double "Twitter client" received unexpected message :update with ("今日は晴れです")
```

## 2. モックに返事の仕方を教え込む
もう一度サンプルコードを見てみる。

```ruby
  def tweet_forecast
    twitter_client.update '今日は晴れです'
  end

  def twitter_client
    Twitter::REST::Client.new
  end
```

`tweet_forecast`メソッドが呼ばれると、メソッド内で`twitter_client.update`を呼び出す。
`twitter_client`の部分が今回のモックに変わる予定なので、モックは`update`というメソッドを呼び出せるようにする必要がある。

その設定をしているのが次のコード
```ruby
allow(twitter_client_mock).to receive(:update)
```

RSpecでは`allow(モックオブジェクト).to receive(メソッド名)`の形で、モックに呼び出し可能なメソッドを設定できる。

また、この状態では`update`以外のメソッドを呼び出すことはできない。
もし呼び出されるとエラーが発生し、テストが失敗する。
他のメソッドも呼ばれる予定になっている場合は、同じ構文を使って呼び出し可能なメソッドを追加していく必要がある。

## 3. アプリケーションコードにモックをこっそり送り込む
アプリケーション側の実装をこっそりモックに置き換える。
それをやっているのが以下の部分
```ruby
weather_bot = WeatherBot.new
allow(weather_bot).to receive(:twitter_client).and_return(twitter_client_mock)
```

もう一度アプリケーション側のコードも載せる

```ruby
def tweet_forecast
  twitter_client.update '今日は晴れです'
end

def twitter_client
  Twitter::REST::Client.new
end
```

テストコードでやったのは、アプリケーション側の`twitter_client`の置き換え。
`twitter_client`メソッドを呼び出されたら、`Twitter::REST::Client.new`ではなく、`twitter_client_mock`を返すように変更した。

構文としては「2. モックに返事の仕方を教え込む」とほぼ同じ

```ruby
allow(実装を置き換えたいオブジェクト).to receive(置き換えたいメソッド名).and_return(返却したい値やオブジェクト)
```

これで、`witter_client`メソッドを呼び出されたときに本来の実装とは異なるオブジェクト(twitter_client_mock)を返すようになった。


## 4. テストしたいメソッドを呼び出す
モック(`twitter_client_mock`)には`update`というメソッドが呼ばれていることを教え、アプリケーション側(`weather_bot`)は本来の実装を置き換えられてモックを使うようになった。
あとは、アプリケーション側のメソッドを呼び出せばOK
```ruby
expect{weather_bot.tweet_forecast}.not_to raise_error
```

weather_botはモックを使うように改造されているので、`twitter_client.update '今日は晴れです'`を実行しても実際にツイートが飛ぶことはない。

## モックのメソッドがちゃんと呼び出されていることを検証する
モックには「セットアップしたメソ度がちゃんと呼び出されたかどうか」を検証する機能がある。

例えば先程のコードを改造して「`update`メソッドが呼び出されていること」も検証してみる

```ruby
it 'エラーなく予報をツイートすること' do
  twitter_client_mock = double('Twitter client')
  # updateメソッドが呼ばれることもあわせて検証する
  expect(twitter_client_mock).to receive(:update)

  weather_bot = WeatherBot.new
  allow(weather_bot).to receive(:twitter_client).and_return(twitter_client_mock)
  expect{ weather_bot.tweet_forecast }.not_to raise_error
end
```

変更したのはこの部分のみ

```ruby
expect(twitter_client_mock).to receive(:update)
```
ここでは`expect`に変わっている。
expectを使うと「そのメソッドが呼び出されないとテスト失敗」になる。

モックを使うときは、単に実装を置き換えたいだけなのか、それともメソッドの呼び出しも研掃したいのかに応じてallowとexpectを使い分ける必要がある。


## 更にモックを使いこなす

### わざとエラーを発生させてエラー処理をテストする。
先程のサンプルコードにエラー処理を追加してみる。
ツイートを発信するときにエラーが発生したら、そのエラーを通知するようにしてみる。


```ruby
class WeatherBot
  def tweet_forecast
    twitter_client.update '今日は晴れです'
  rescue => e
    notify(e)
  end

  def twitter_client
    Twitter::REST::Client.new
  end

  def notify(error)
    # （エラーの通知を行う。実装は省略）
  end
end
```

次にテストコードを書いていく
ここでは「エラーが起きたら`notify`メソッドが呼ばれること」を検証している。

```ruby
it 'エラーが起きたら通知すること' do
  twitter_client_mock = double('Twitter client')
  # updateメソッドが呼ばれたらエラーを発生させる
  allow(twitter_client_mock).to receive(:update).and_raise('エラーが発生しました')

  weather_bot = WeatherBot.new
  allow(weather_bot).to receive(:twitter_client).and_return(twitter_client_mock)
  # notifyメソッドが呼ばれることを検証する
  expect(weather_bot).to receive(:notify)

  # tweet_forecastメソッドを呼び出す
  # weather_botのnotifyメソッドが呼び出されたらテストはパスする
  weather_bot.tweet_forecast
end
```

まず注目したいのはこのコード

```ruby
allow(twitter_client_mock).to receive(:update).and_raise('エラーが発生しました')
```

最初のテストコードでは`twitter_client_mock`は`update`メソッドを呼び出されても何もしなかったが、ここではエラーを発生させるようにした。

`allow(オブジェクト).to receive(メソッド名).and_raise(エラー)`の形でメソッドが呼ばれたときに、人為的にエラーを発生させることができる。

次に注目したいのはこのコード

```ruby
expect(weather_bot).to receive(:notify)
```

エラー処理が適切に行われれば`weather_bot`の`notify`メソッドが呼ばれるはず。
そこで`notify`メソッドをモック化して、このメソッドがちゃんと呼ばれている事を検証している。
最後に`weather_bot.tweet_forecast`を呼び出して、アプリケーションコードを実行している。

# 参考資料
[使えるRSpec入門・その1「RSpecの基本的な構文や便利な機能を理解する」](https://qiita.com/jnchito/items/42193d066bd61c740612)

