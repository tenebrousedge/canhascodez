---
title: "How Not to Code"
date: 2019-04-17T18:45:00-07:00
summary: "Write better code by avoiding these simple mistakes." 
featuredImage: "images/markus-spiske-507983-unsplash.jpg"
imageCredit: <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@markusspiske?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Markus Spiske"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Markus Spiske</span></a>
---


## Things That Make Kais 😭

This is a topic that I feel extremely strongly about, but I think that I'm going to try to write a useful rather than cathartic post here. These are some things that you should always avoid in your code.

### `each`

In other languages, `each` or `forEach` might be an accepted idiom. It seems like it would be a generically useful iterator. The problem is that it does not have a useful return value: the loop cannot affect the program state without interacting with something outside the loop.

Bad:
```ruby
array = []
(0..9).each { |i| array << i ** 2 }
array
```

Good:
```ruby
(0..9).map { |i| i ** 2 }
```

As shown, if you want to create a new value with an `each` loop, you must declare a temporary value elsewhere, reach outside the loop's scope, and expend another line after the loop to do something useful with that return value. There are some cases where creating a new value is not important. If you are creating a higher-level iterator, such as `map` or `reduce`, then you don't really have an alternative but to use the lower-level iterator. If you are doing metaprogramming, then you probably don't care that `define_method` has a return value. There are a couple other similar situations, but broadly, if you can use a different iterator, do it. And no, I do not mean that you should use `times`. It has the same problems.

### `while` and `until`

Again, this is a low-level iterator which has more specialized uses than it appears. You should **never** use either `while` or `until` when the number of iterations is knowable. If possible, these methods should be wrapped in `Enumerator` or `Iterator` (Crystal), so that they can be treated as enumerable collections.
```crystal
class Collatz
  include Iterator(Int32)

  def initialize
    @current = 0
  end

  def next
    @current += 1
    collatz(@current)
  end

  def collatz(n)
    steps = 0
    until n == 1
      n.even? ? (n /= 2) : (n = 3 * n + 1)
      steps += 1
    end
    steps
  end
end
```

### `for`

<s>There are zero reasons to ever use `for` loops.</s>

Okay, there seems to be an exception to this rule. There is some sense in which you can use higher-order iterators in Crystal's [metaprogramming][crystal-metaprogramming], but it can be easiest to use a `for` loop. This may change in the future.

[crystal-metaprogramming]: https://stackoverflow.com/questions/48037084/why-does-crystals-macro-syntax-for-iterating-differ-from-the-rest-of-crystal
