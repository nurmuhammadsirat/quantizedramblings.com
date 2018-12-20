---
title: Using instance variables in class methods in Ruby
asset_path: /assets/using-instance-variables-in-class-methods-in-ruby
updated: 2017-09-29 23:48
---

I was looking at some legacy Ruby code today and saw that there was a bunch of instance variables being created inside a class method. In fact, the whole class is littered with `def self.foo` declarations; nothing except class methods. There was an even funky statement at the top of the class.

```Ruby
class Foo
  class << self
    attr_reader :instance_var_1, :instance_var_2, :instance_var_3
  end

  ...

end
```

Needless to say, I was intrigued. Who would write Ruby code this way? What was the intent and motivation? Was it laziness or awesomeness? Was there some intricacies of Ruby that I was not aware of? That must be it. Faced with something I know I don't know, naturally, I wanted to experiment. Off to the REPL I say..

```Ruby
class Foo
  def self.bar=(val)
    @bar = val
  end

  def self.bar
    @bar
  end
end
```

So I created my own dummy class to work with. Standard enough getter/setter but using a class method.

```Ruby
Foo.bar = "hello world"
=> "hello world"

Foo.bar
=> "hello world"
```

Works as expected. Now to unknown territory.

```Ruby
a = Foo.new
=> #<Foo:0x007f95ab80d5e8>

b = Foo.new
=> #<Foo:0x007f95aa17e688>

a == b
=> false
```

Again, as expected.

```Ruby
a.class.bar
=> "hello world"

b.class.bar
=> "hello world"
```

Expected.

```Ruby
a.class.bar = 12345
=> 12345
```

So what happens when I call #bar from b?

```Ruby
b.class.bar
=> 12345
```

Interesting! Since Foo is an instance of Class, the instance variable @bar lives on that instance. Any sub-instances we create from Foo and then subsequently attempt to access @bar will look at the same reference.

So the moral of the story: while it is fine to use instance variables in class methods, you probably shouldn't.
