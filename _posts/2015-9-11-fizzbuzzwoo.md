---
layout: post
title: fizzbuzzwoo
---

Most methods of coding `fizzbuzz` rely on conditional statements that make it hard to generalise the algorithm to cope with new rules. This raises the question of how to code the function without using conditionals at all.

One approach is to simply mimic the Boolean algebra of conditional statements using integer arithmetic. But this would be cheating. The underlying logic of the algorithm is unchanged by hijacking integer methods in this fashion. We would just masking our use of conditionals with fancy tricks. The code would if anything become even less flexible.

If we want flexibility, let's try coding `fizzbuzzwoo` – which is like `fizzbuzz` but with an additional rule invoking `'woo'` for every multiple of `7`. We do with without any kind of conditional branching. Instead we set up a hash of procs representing our substitution rules, and use this hash to transform `n` when required.

Let's start with a hash populated by prime keys and corresponding string values:

```
prime_factors = { 3=>'fizz', 5=>'buzz', 7=>'woo' }
```

We can extend this hash by adding in all the composite factors and compound strings:

```
factors = { 3=>'fizz', 5=>'buzz', 7=>'woo', 15=>'fizzbuzz',
            21=>'fizzwoo', 35=>'buzzwoo', 105=>'fizzbuzzwoo' }
```

Note that `factors[n.gcd(105)]` returns the appropriate string for every `n` that has `3`, `5` or `7` as a factor. But it doesn't work if `n` and `105` are coprime: `factors[143.gcd(105)]`, for instance, returns `nil` rather `143`. Clearly we need to add the key `1` to our hash – but what should its value be?

In these comprime cases we want to leave `n` alone rather than replace it with a string. And that involves a function dependent upon on `n`, rather than a fixed string that does not 

We deal with this problem by first converting each value in our hash from string to a proc:

```
factor_procs = { 3=>proc{'fizz'}, 5=>proc{'buzz'}, 7=>proc{'woo'},
                 15=>proc{'fizzbuzz'}, 21=>proc{'fizzwoo'},
                 35=>proc{'buzzwoo'}, 105=>proc{'fizzbuzzwoo'} }
```

Then we store the identity function in the hash as a proc with key `1`:

```
factor_procs[1] = proc{|x| x}
```

At last we can define `fizzbuzzwoo(n)` to be `factor_procs[n.gcd(105)].call(n)`

Putting this all together in Ruby code we get:

```
def fizzbuzzwoo(n)

  factor_procs = { 1=>proc{|x| x}, 3=>proc{'fizz'}, 5=>proc{'buzz'},
                   7=>proc{'woo'}, 15=>proc{'fizzbuzz'}, 21=>proc{'fizzwoo'},
                   35=>proc{'buzzwoo'}, 105=>proc{'fizzbuzzwoo'} }

  factor_procs[n.gcd(105)].call(n)

end
```

Then `(1..105).map {|n| fizzbuzzwoo(n)}` returns:

```
[1, 2, 'fizz', 4, 'buzz', 'fizz', 'woo', 8, 'fizz', 'buzz', 11, 'fizz', 13, 'woo',
'fizzbuzz', 16, 17, 'fizz', 19, 'buzz', 'fizzwoo', 22, 23, 'fizz', 'buzz', 26, 'fizz', 'woo',
29, 'fizzbuzz', 31, 32, 'fizz', 34, 'buzzwoo', 'fizz', 37, 38, 'fizz', 'buzz', 41, 'fizzwoo',
43, 44, 'fizzbuzz', 46, 47, 'fizz', 'woo', 'buzz', 'fizz', 52, 53, 'fizz', 'buzz', 'woo',
'fizz', 58, 59, 'fizzbuzz', 61, 62, 'fizzwoo', 64, 'buzz', 'fizz', 67, 68, 'fizz', 'buzzwoo',
71, 'fizz', 73, 74, 'fizzbuzz', 76, 'woo', 'fizz', 79, 'buzz', 'fizz', 82, 83, 'fizzwoo',
'buzz', 86, 'fizz', 88, 89, 'fizzbuzz', 'woo', 92, 'fizz', 94, 'buzz', 'fizz', 97, 'woo',
'fizz', 'buzz', 101, 'fizz', 103, 104, 'fizzbuzzwoo']
```
