# what
- TDD(テスト駆動開発)の基本的な流れについてRSpecを利用して体験してみる

# 作業環境
- Ruby: ver 2.6
- RSpec: ver 3.10

ディレクトリ構造は以下を想定
```
.
└── lib
    ├── hogehoge.rb
    └── huga.rb
└── spec
    ├── hogehoge_spec.rb
    └── huga.rb
    └── spec_helper.rb
```


# ステップ1. 必要ファイルを作成する
テストコードとプロダクションコードを書くためのファイルをそれぞれ作成する
この段階ではどちらのファイルもまだ中身は書かない
今回は`dog.rb`, `dog_spec.rb`という名前でファイルを作成する

```
$ touch lib/dog.rb
$ touch spec/dog_spec.rb
```

# ステップ2. テストコードの中身を書いていく
先に`dog_spec.rb`のテストコードを書いていく
とっても内容的にはテストを定義するものだけ

```ruby
require "spec_helper"

Rspec.describe Dog do
  it "it named 'Pochi'"
end
```

テストを実行してみる
```
$ bundle exec rspec

An error occurred while loading ./spec/dog_spec.rb.
Failure/Error:
  RSpec.describe Dog do
    it "it named 'Pochi'" do
    end
  end

NameError:
  uninitialized constant Dog
# ./spec/dog_spec.rb:3:in `<top (required)>'
No examples found.

Top 0 slowest examples (0 seconds, 0.0% of total time):

Finished in 0.00002 seconds (files took 0.10768 seconds to load)
0 examples, 0 failures, 1 error occurred outside of examples
```

「Dogなんてものないよ」と言われている
(dog.rbの方には中身を書いていないので当たり前といえば当たり前である)

# ステップ3. プロダクションコードに中身を書いていく
dog.rbに中身を書いていく

```ruby
class Dog
end
```

テストコードの中身は変えずにテストを実行する

```
$ bundle exec rspec

Randomized with seed 48632

Dog
  it named 'Pochi' (PENDING: Not yet implemented)

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Dog it named 'Pochi'
     # Not yet implemented
     # ./spec/dog_spec.rb:4


Top 1 slowest examples (0.00001 seconds, 0.2% of total time):
  Dog it named 'Pochi'
    0.00001 seconds ./spec/dog_spec.rb:4

Finished in 0.00486 seconds (files took 0.0876 seconds to load)
1 example, 0 failures, 1 pending

Randomized with seed 48632
```

今回はクラス名を書いただけで名前は書いていないので、ペンディング状態となる

# ステップ4. 期待通りに落ちるテストを書く
dog.rbはそのままで、dog_spec.rbに内容を追加していく。
プロダクションコードには中身が無いのでテストを実行すると失敗するハズ

```ruby
require_relative "../lib/dog"

RSpec.describe Dog do
  it "it named 'Pochi'" do
    dog = Dog.new
    expect(dog.name).to eq 'Pochi'
  end
end
```

テストを実行してみる

```
$ bundle exec rspec

Randomized with seed 20218

Dog
  it named 'Pochi' (FAILED - 1)

Failures:

  1) Dog it named 'Pochi'
     Failure/Error: expect(dog.name).to eq 'Pochi'

     NoMethodError:
       undefined method `name' for #<Dog:0x00007ffbf830e728>
     # ./spec/dog_spec.rb:6:in `block (2 levels) in <top (required)>'

Top 1 slowest examples (0.00026 seconds, 3.9% of total time):
  Dog it named 'Pochi'
    0.00026 seconds ./spec/dog_spec.rb:4

Finished in 0.0067 seconds (files took 0.17321 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/dog_spec.rb:4 # Dog it named 'Pochi'

Randomizedwith seed 20218
```

想定通り失敗した


# ステップ5.  テストが通るように実装する
dog.rbの中身を追記する

```ruby
class Dog
  attr_accessor :name

  def initialize(name="Pochi")
    @name = name
  end
end
```

テストを実行する

```
$ bundle exec rspec

Randomized with seed 48141

Dog
  it named 'Pochi'

Top 1 slowest examples (0.00702 seconds, 53.9% of total time):
  Dog it named 'Pochi'
    0.00702 seconds ./spec/dog_spec.rb:4

Finished in 0.01301 seconds (files took 0.08564 seconds to load)
1 example, 0 failures

Randomized with seed 48141
```

今度はテストが通った


# 参考
[はじめてのRSpec - まずテスト書いてからコード書くシンプルなチュートリアル](https://qiita.com/luckypool/items/e3662170033347510c3c)
