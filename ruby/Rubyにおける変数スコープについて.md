Rubyの変数には、その変数を参照したり変更できる範囲(スコープ)が設定されている。
範囲の外からはその変数を利用できない。
今回は、以下の変数を対象に変数スコープを確認する

<今回の対象>
- クラス変数
- インスタンス変数
- ローカル変数
- グローバル変数

## クラス変数

### 定義の仕方
`@@変数名`で定義する。
クラス定義内・クラスメソッド内・initializeメソッド内・インスタンスメソッド内でも定義できる。

```ruby
@@class_name = xxxx
```

### スコープ
以下の範囲内で参照が可能
 - インスタンスメソッド
 - クラスメソッド
 - クラス定義内
 - initializeメソッド

クラス内であればどこでも参照・定義・更新できる
ただし、クラスを超えた参照はできない（継承が必要）

```ruby
class Num
  # クラス変数を定義
  @@number = 4

  # インスタンスメソッド内でクラス変数を呼び出す
  def from_instance_method
    p @@number
  end

  # クラスメソッド内でクラス変数を呼び出す
  def self.from_class_method
    p @@number
  end

  # initializeメソッド内でクラス変数を呼び出す
  def initialize
    p @@number
  end

  # クラス定義内でクラス変数を呼び出す
  p @@number
end

# インスタンスを作成(initializeメソッドを実行)
num = Num.new

# インスタンスメソッドを実行
num.from_instance_method

# クラスメソッドを実行
Num.from_class_method
```

実行結果

```
$ bundle exec ruby scope_1.rb
4
4
4
4
```

## インスタンス変数

### 定義の仕方
`@変数名`で定義する。
インスタンスメソッドかinitializeの中で定義する。
クラス定義内で定義すると、クラスインスタンス変数という別の変数になり動作が異なる。

```ruby
@variable_name = xxxx
```

### スコープ
以下の範囲内で参照・更新が可能
- インスタンスメソッド
- initializeメソッド

```ruby
class Car
  # initializeメソッド内でインスタンス変数を定義+呼び出し
  def initialize
    @car_name = "lexus"
    p @car_name
  end

  # インスタンスメソッド内でインスタンス変数を再定義+呼び出し
  def from_instance_method
    @car_name = "prius"
    p @car_name
  end

  # クラスメソッド内でインスタンス変数を呼び出す
  def self.from_class_method
    p @car_name
  end

  # クラス定義内でインスタンス変数を呼び出す
  p @car_name # => nil 失敗
end

# インスタンスを作成(initializeメソッドを実行)
car = Car.new # => "lexus" 成功

# インスタンスメソッドを実行
car.from_instance_method #=> "prius" 成功

# クラスメソッドを実行
Car.from_class_method #=> nil 失敗
```

実行結果

```
$ bundle exec ruby scope_2.rb
nil
"lexus"
"prius"
nil
```

インスタンスメソッドとinitializeメソッドの外では利用できない様子。

## クラスインスタンス変数
※ 正しくはClasssクラスのオブジェクトが持つインスタンス変数らしい
[RubyのClassクラスについてとか、クラスを知るためのメソッドとか、あれこれ（メモ）](http://bambinya.hateblo.jp/entry/2016/01/10/151216)

見た目はインスタンス変数だが、機能はクラス変数に近い。
クラス変数との違いは
- 継承しても利用できない
- インスタンスメソッドから利用できない。


```ruby
class Car
  # クラス定義内に変数を定義
  @car_name = "lexus"

  # クラスメソッド内で変数を呼び出し
  def self.from_class_method
    p @car_name
  end

  # initializeメソッド内で変数を呼び出す
  def initialize
    p @car_name
  end

  # インスタンスメソッド内で変数を呼び出す
  def from_instance_method
    p @car_name
  end

  # クラス定義内から呼び出し
  p @car_name  #=> "lexus" 成功
end

# クラスメソッドを実行
Car.from_class_method #=> "lexus" 成功

# インスタンスを作成(initializeメソッドを実行)
car = Car.new #=> nil 失敗

# インスタンスメソッドを実行
car.from_instance_method #=> nil 失敗

# クラスを継承させる
class Bike < Car
end

# 継承先のクラスでクラスメソッドを実行
Bike.from_class_method #=> nil (変数呼び出し)失敗
```

実行結果
```
$ bundle exec ruby scope_3.rb
"lexus"
"lexus"
nil
nil
nil
```

## ローカル変数
### 定義の仕方
@はなく、変数名のみで定義する。

```
variable_name = xxxx
```

### スコープ
以下の範囲内で参照・更新が可能
- 定義した位置からその変数が定義されたブロック、メソッド、またはクラス/モジュール定義の終わりまで
    - 定義された場所の中だけ使える。メソッドをまたいだりすると使えない。

```ruby
class Car
  # クラス定義内にローカル変数を定義①
  car_name = "lexus"

  # クラスメソッド内で①を呼び出し+ローカル変数を定義②
  def self.from_class_method
    p car_name # 呼び出し失敗
    class_defined = "at class"
  end

  # initializeメソッド内で①を呼び出し+ローカル変数を定義③
  def initialize
    p car_name # 呼び出し失敗
    initialize_defined = "at initialize"
  end

  # インスタンスメソッド内で①②③を呼び出し+ローカル変数を定義④
  def from_instance_method
    p car_name # 呼び出し失敗
    p class_defined # 呼び出し失敗
    p initialize_defined # 呼び出し失敗
    instance_defined = "at_instance"
  end

  # 別のインスタンスメソッド内で④を呼び出し⑤
  def fom_another_instance_method
    p instance_defined #呼び出し失敗
  end

  # クラス定義内から呼び出し
  p car_name # 成功
  p class_defined # 呼び出し失敗
  p initialize_defined # 呼び出し失敗
  p instance_defined # 呼び出し失敗

  [1,2,3].each{p(car_name)} # 成功
end
```

- クラス定義内で定義したローカル変数はクラスメソッド、インスタンスメソッド内で利用できない
- クラスメソッド、インスタンスメソッド内で定義したローカル変数は外のメソッドは利用できない


## グローバル変数
### 定義の仕方
戦闘に`$+変数名`で定義する

```ruby
$variable_name = xxxx
```

### スコープ
プログラムのどこからでも参照ができる。

```ruby
# グローバル変数用のスコープ確認
class Car
  # クラス定義内に変数を定義
  $car_name = "lexus"

  # クラスメソッド内で変数を呼び出し
  def self.from_class_method
    p $car_name
  end

  # initializeメソッド内で変数を呼び出す
  def initialize
    p $car_name
  end

  # インスタンスメソッド内で変数を呼び出す
  def from_instance_method
    p $car_name
  end

  # クラス定義内から呼び出し
  p $car_name
end

# クラスメソッドを定義
Car.from_class_method

# インスタンスを作成
car = Car.new

# インスタンスメソッドを実行
car.from_instance_method

# クラスを継承させる
class Bike < Car
end

# 継承先のクラスでクラスメソッドを実行
Bike.from_class_method
```

実行結果
```
$ bundle exec ruby scope_5.rb
"lexus"
"lexus"
"lexus"
"lexus"
"lexus"
```
