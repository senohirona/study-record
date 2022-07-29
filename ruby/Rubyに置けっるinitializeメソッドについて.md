# what
rubyにおけるinitializeメソッドについて、理解したことなどを記載する


# initialalizeメソッドとは
クラスにはinitalizeという特別なメソッドが用意されている。
initializeという名前のメソッドを作ると、オブジェクトが新しく作られる時に自動で呼び出される。

```ruby
class Drink
    def initialize
        puts "新しいオブジェクト"
    end
end

Drink.new
```

initializeは他のメソッドと同じ手順で定義して、名前をinitializeにするだけ。
そして`.new`メソッドが呼ばれるとオブジェクトが作られるが、この際にinitalizeメソッドが自動で呼ばれる。
**注意**
呼び出しているのはnewメソッドだが、自動で呼び出されるのはinitalizeメソッドの方。

## 基本的な挙動

> このメソッドは Class#new から新しく生成されたオブジェクトの初期化のために呼び出されます。他の言語のコンストラクタに相当します。デフォルトの動作ではなにもしません。

クラスメソッドのように`クラス名.メソッド名`という呼び方をしてもinitializeは実行されない。

```ruby
class Car
  def initialize
    puts "initalize car name"
    @car_name = "lexus"
  end

  class << self
    def car_name
      p @car_name
    end
  end
end

# オブジェクトを生成して呼び出す
car = Car.new
p car

# クラスメソッドから呼び出す
Car.car_name
```

実行結果

```
$ bundle exec ruby scope_initialize.rb
initalize car name
#<Car:0x00007fb7628e38e0 @car_name="lexus">

# ↓クラスメソッドを呼び出したときの結果（initializeで設定した値は入っていない)
nil
```

# 参考
[instance method Object#initialize](https://docs.ruby-lang.org/ja/latest/method/Object/i/initialize.html)
