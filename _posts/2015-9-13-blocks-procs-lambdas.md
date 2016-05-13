---
layout: post
title: blocks, procs, lambdas
---

## blocks

A **block** in Ruby is simply a sequence of code enclosed by `{` and `}` (or alternatively enclosed by `do` and `end`). You can optionally supply a block with local variables by enclosing them with `|` and `|` at the start of the block.

Here are some examples of blocks:

```
{ print "foo" }         # prints "foo"
{ |x| x * 2 + 1 }       # doubles x then adds 1
{ |x, y| x * y + 1 }    # multiplies x and y then adds 1
```

Blocks are NOT objects, but merely syntactical constructions in Ruby. You cannot assign them to variables, or store them in arrays. Pretty much all you can do with them is pass them to methods. Certain methods in Ruby expect a block to be passed to them.

Here are some examples of passing blocks to methods:

```
3.times { print "foo" }             # prints "foofoofoo"
[1, 2, 3].map { |x| x * 2 + 1 }     # returns [3, 5, 7]
```

When defining a method we use the `yield` command (possibly with parameters) to execute the block it was given:

``` 
def hello(a, b)
  print "hello"
  yield(a, b)
end

hello(3, 5) { |x, y| x * y + 1 }    # prints "hello", returns 16
hello(7, 2) { |x, y| x ** y - 1 }   # prints "hello", returns 48
```


## procs

Blocks execute extremely quickly, but are otherwise rather limited. We cannot pass more than one block to a method. Nor can we store blocks: if we want to use them same block in more than one place we have to type it out again.

We have to turn blocks into objects of some sort if we want to do anything more complex with them. This comes with a performance overhead, however, so we should only do it when necessary.

A **proc** is simply a block that has been converted into an object. They are instances of the class `Proc` which inherits from `Object`. They work much like any other object in Ruby – we can instantiate them with `Proc.new` (or its synonym `proc`) and execute them with the instance method `call`:

```
my_proc = Proc.new { |x| x * 2 + 1 }

my_proc.call(7)    # returns 15
my_proc.call(23)   # returns 47
```

We can convert back and forth between blocks and procs by using the unary `&` operator. If `fred` is a proc, `&fred` supplies the corresponding block:

```
fred = proc { print "zap" }
judy = proc { |x| x + 7 }

3.times &fred         # prints "zapzapzap"
[1, 2, 3].map &judy   # returns [8, 9, 10]
```

If `fred` is not a proc, Ruby evaluates `&fred` by calling `fred.to_proc` first and then converting the resulting proc into a block. We often see this used with symbols, since calling `to_proc` on a symbol returns the method that symbol names as a proc:

```
["a", "b", "c"].map &:ord     # returns [97, 98, 99]
```

We can also use the `&` operator the other way round: to convert blocks into procs in method definitions. If the last argument in a method definition is `&judy`, Ruby looks for a block whenever that method is called, then turns the block into a proc named `judy`:

```
def my_method(a, b, &judy)
  print judy.arity
  judy.call(a, b)
end

my_method(3, 5) { |x, y| x * y + 3 }    # prints 2, returns 18
```


## lambdas

A **lambda** is a special type of proc whose behaviour is modified to more closely resemble mathematical functions. The name "lambda" derives from the [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus), a formal system for specifying functions developed by Alonzo Church in the 1930s.

Ruby is pretty relaxed about the number of arguments passed to a block or proc. If there are too many, it simply ignores the extra ones. If there are too few, it doesn't mind unless it actually needs the missing arguments:

```
three = proc { |x| 3 }

three.call        # returns 3
three.call(0)     # returns 3
three.call(0,0)   # returns 3
```

Mathematically, however, the constant 3, the function x ↦ 3 and the function (x, y) ↦ 3 are all quite distinct – they have different domains, despite always giving the same result. Lambdas behave more like maths than code:

```
three = lambda { |x| 3 }

three.call        # raises "wrong number of arguments" error
three.call(0)     # returns 3
three.call(0,0)   # raises "wrong number of arguments" error
```

Lambdas also behave differently when they encounter a `return` command. The Ruby interpreter simply goes back to the method that called the lambda and continues running that method:

```
def square(f)
  y = f.call
  y * y
end

five = lambda { return 5 }

square(five)      # returns 25
```

In contrast, if you call a proc (or yield to a block) inside a method, Ruby tries to exit the whole method on encountering a `return`. This will raise a local jump error if the proc was defined externally:

```
def square(f)
  y = f.call
  y * y
end

five = proc { return 5 }

square(five)      # raises "unexpected return" error
```

It will simply exit the method immediately if the proc was defined inside it:

```
def square_five
  five = proc { return 5 }
  y = five.call
  y * y
end

square_five      # exits at proc call and returns 5
```

Contrast the behaviour if we use a lambda instead:

```
def square_five
  five = lambda { return 5 }
  y = five.call
  y * y
end

square_five      # exits at end of method and returns 25
```
