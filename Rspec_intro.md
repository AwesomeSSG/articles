[Rspec](https://github.com/rspec/rspec-core)是ruby的测试框架之一。

[Mock](http://www.mockobjects.com/)和[stub](https://en.wikipedia.org/wiki/Method_stub)都属于[Test double](http://www.martinfowler.com/bliki/TestDouble.html)，用于测试时，模拟特定的方法或者对象的值或行为。

在Rspec中提供了这3种方法（gem rspec-mocks）。

## Test Double
---
Test Double是一个模拟对象，在代码中模拟系统的另一个对象，方便测试。

例如我们有个智能书架（Bookshelf），可以自动统计书（book）的总重量（weight_of_books）。在测试这个方法的时候，需要我们有book这个对象，而Book类仍在开发中，不能被我们使用。

```rb
book = double("Book")
```
这样我们就有了一个book对象. 但现在book没有任何方法，这是空的类，我们可以使用stub等方法为他创建假方法。

同时，也可以直接创建一个有属性的book对象。

```rb
book = double("Book", weight: 80)
```

```rb

book1 = double("Book", weight: 80)
book2 = double("Book", weight: 20)


bookshelf = Bookshelf.new

bookshelf.add(book1)
expect(bookshelf.weight_of_books).to eq(80)

bookshelf.add(book2)
expect(bookshelf.weight_of_books).to eq(100)

```
这样，我们就测试了bookshelf的方法。

**instance_double** 是RSpec 3之后替代double，并支持verifying double，意思是当使用stub的方法后，RSpec还会去验证被double的真实对象是否具有这个方法。

## Method Stubs
---

Stub 既可以模拟假对象（test double）的方法，也可以模拟真对象（real object）的方法。RSpec－mock提供下面3种创建method stub的方法。

```rb

allow(book).to receive(:title) { "The RSpec Book" } # 第一个参数（book）是对象，第二个参数是方法， 第三个参数是方法的返回值
allow(book).to receive(:title).and_return("The RSpec Book")
allow(book).to receive_messages(
    :title => "The RSpec Book",
    :subtitle => "Behaviour-Driven Development with RSpec, Cucumber, and Friends")

```
上面等同于：

```rb
book = double("book", :title => "The RSpec Book" )
```

#### stub返回多个值

```rb
allow(die).to receive(:roll).and_return(1, 2, 3)
die.roll # => 1
die.roll # => 2
die.roll # => 3
die.roll # => 3
die.roll # => 3
```
每次调用stub的方法，会按顺序依次返回。

##  Mock
---
Mock与Stub的区别在于， Mock是对象层面的，Stub是方法层面。Mock对象同时支持method的stub和 消息验证（message expectation）

通常Rspec使用expect去验证message是否工作。

```rb
validator = double("validator")
expect(validator).to receive(:validate) { "02134" }
zipcode = Zipcode.new("02134", validator)
zipcode.valid?
```
如果validate消息在用例执行到最后有被接收到，则验证成功，否则失败。

## 新老版本差异
---
Rspec 老新语法差异（前为老，后为新），前后的作用基本一样：

- *stub* **vs** *allow*
- *should_recevice* **vs** *expect().to recevice*
- *any_instance* (已遗弃)
- *stub_chain* （已遗弃）
- *unstub* **vs** *and_call_original*


#### 1. *stub* **vs** *allow*

对于`should` 方法做断言，通常对应的stub方法：

```rb
describe "a stubbed implementation" do
  it "works" do
    object = Object.new
    object.stub(:foo) do |arg|
      if arg == :this
        "got this"
      elsif arg == :that
        "got that"
      end
    end

    object.foo(:this).should eq("got this")
    object.foo(:that).should eq("got that")
  end
end
```

对于RSpec3的 `expect`方法，对应的stub方法：

```rb
describe "a stubbed implementation" do
  it "works" do
    object = Object.new
    allow(object).to receive(:foo) do |arg|
      if arg == :this
        "got this"
      elsif arg == :that
        "got that"
      end
    end

    expect(object.foo(:this)).to eq("got this")
    expect(object.foo(:that)).to eq("got that")
  end
end
```

#### 2. *should_recevice* **vs** *expect().to recevice*

老版本的Rspec（2.14）， 使用should_receive. RSpec3使用了`expect`

```rb
Session.should_receive(:get).with('test_session_id').and_return(mock_session)
```

```rb
expect(Session).to receive(:get).with('test_session_id').and_return(mock_session)
```

#### 3. *any_instance* (已遗弃)

```rb
Incident.any_instance.stub(:field_definitions).and_return([])
```
意思是Incident类的所有实例都具备stub的方法，不过该方法已经被遗弃。

```rb
RSpec.describe "Expecting a message on any instance of a class" do
  before do
    Object.any_instance.should_receive(:foo)
  end

  it "passes when an instance receives the message" do
    Object.new.foo
  end

  it "fails when no instance receives the message" do
    Object.new.to_s
  end
end
```
上述结果：

`2 examples, 1 failure`
`Exactly one instance should have received the following message(s) but didn't: foo`

任意new的Object类的实例都会有foo方法，当使用`should_recevice`时，所有的Object的实例都必须响应foo方法，否则就会报错。

#### *stub_chain* （已遗弃）

支持链式方法的stub，现在已经不推荐使用。

```rb
  example "using a string and a block" do
    dbl.stub_chain("foo.bar") { :baz }
    expect(dbl.foo.bar).to eq(:baz)
  end

```

#### *unstub* **vs** *and_call_original*


`unstub`将之前实例stub的方法disable掉，声明之后，stub的方法，将被原有的方法替代或者不存在。

```rb
RSpec.describe "Unstubbing a method" do
  it "restores the original behavior" do
    string = "hello world"
    string.stub(:reverse) { "hello dlrow" }

    expect {
      string.unstub(:reverse)
    }.to change { string.reverse }.from("hello dlrow").to("dlrow olleh")
  end
end
```

`and_call_original` 是类似的功能，使用`with`可以让特定的参数输入时，仍然适用stub的方法返回值

```rb
require 'calculator'

RSpec.describe "and_call_original" do
  it "can be overriden for specific arguments using #with" do
    allow(Calculator).to receive(:add).and_call_original
    allow(Calculator).to receive(:add).with(2, 3).and_return(-5)

    expect(Calculator.add(2, 2)).to eq(4)
    expect(Calculator.add(2, 3)).to eq(-5)
  end
end
```

## 配置返回方式

以下是 RSpec 3.4 版本特性

- **and_return**   返回值
- **and_raise**    抛出异常
- **and_throw**    抛出symbol
- **and_yield**    让方法接受块参数
- **and\_call_original**  调用原有的方法
- **and\_wrap_original**  使用块修改原有的方法

具体见： [Configuring responses](http://www.relishapp.com/rspec/rspec-mocks/v/3-4/docs/configuring-responses)



## References

[instance_double and verifying double](http://rhnh.net/2013/12/10/new-in-rspec-3-verifying-doubles#fn1)

[rspec-verifying-doubles](http://wegowise.github.io/blog/2014/09/03/rspec-verifying-doubles/)

[rspec mock 2.14](http://www.relishapp.com/rspec/rspec-mocks/v/2-14/docs/method-stubs)

[Test double](http://xunitpatterns.com/Test%20Double.html)
