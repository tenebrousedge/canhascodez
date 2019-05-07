---
title: "Exercism № 1: Hamming"
date: 2019-03-17T18:49:54-07:00
draft: true
tags: ["ruby", "crystal", "exercism"]
categories: ["programming"]
summary: "We begin exploring Ruby and Crystal with Exercism problems, starting with counting differences between strings."
featuredImage: "images/philip-estrada-12529-unsplash-minified.jpg" 
imageCredit: '<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@philipestrada?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Philip Estrada"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Philip Estrada</span></a>'
---

## Counting Similarities and Differences 

The first real Ruby/Crystal [Exercism](https://exercism.io) problem is called "Hamming", and putatively refers to [Hamming codes](https://en.wikipedia.org/wiki/Hamming_code), but really just involves counting differences in two strings. One can iterate over the strings a number of different ways. This is one of the more elegant:

```ruby
module Hamming
  extend self
  
  def compute(strand_1, strand_2)
    raise ArgumentError.new unless strand_1.size == strand_2.size
    strand_1.each_char.zip(strand_2.each_char).count do |(first, second)|
      first != second
    end
  end
end
```

This is also valid Crystal code. We use a "guard clause" to check whether the arguments are of equal length (using `unless` because anything Ruby-specific must be *The Ruby Way*™), and using `each_char` because unlike `chars`, it does not create a new array (just an `Enumerator`). `zip` gives us a new array which associates the elements of the input arrays at each index, which these languages will destructure for us in the block signature for `count`, and the resulting block body is simplicity itself. As mentioned, there are a number of alternatives, most of which are written identically in both languages.  

### Benchmarking

Now, you might be asking yourself, &ldquo;If there is more than one way to do it, which way is fastest?&rdquo; Whenever you start to think this, stop. 97% of the time, this is a bad idea. Your first priority is to write clear and understandable code. When performance cannot be ignored, get a performance profile before you start tweaking. Keep in mind, benchmark results can vary depending on everything from the order the benchmarks are run in to the phase of the moon. In most cases you will get less than an order of magnitude from writing a method one way or another, whereas a more clever algorithm or heuristic has greater potential for order-of-magnitude improvements.

```ruby
# (...)
def compute(first, second)
  raise ArgumentError.new unless first.size == second.size
  (0...first.length).count { |idx| first[idx] != second[idx] }
end
```

We skip creating an intermediate array, and instead create a `Range`, which just stores the first and last elements, and whether the range is exclusive. Once again, there isn't a real alternative to `count`. &ldquo;But wait,&rdquo; say you, &ldquo;This method doesn't transform the strings at all! Shouldn't that be faster?&rdquo; That's exactly the kind of thinking that gets you into trouble! The difference between the two is insignificant. Code doesn't become faster just because you use an index variable. More than likely, the elegant path will also be the fastest. Let's take a look at one more solution:

### Transpose‽

This is an interesting approach, but it's somewhat slow, and probably counterintuitive. I would probably not recommend actually using this technique in real-world code.

Ruby:
```ruby
def compute(*strands)
  strands.map(&:chars)
    .transpose
    .count { |arr| arr.all?(&arr.first.method(:==)) }
rescue IndexError
  raise ArgumentError
end
```

Crystal:
```crystal
def compute(*strands)
  strands.to_a.map(&.chars)
    .transpose
    .count { |arr| !arr.all?(arr.first) }
rescue IndexError
  raise ArgumentError.new
end
```
Whether or not it makes sense for this code to take an arbitrary number of arguments is beyond the scope of this problem, but it's interesting to look at how this may be generalized. Ruby and Crystal both use the splat `*` operator to allow a method to receive an arbitrary number of arguments, but in Crystal this produces a [`Tuple`](https://crystal-lang.org/api/0.27.2/Tuple.html) rather than an `Array`, thus forcing us to use `to_a` to get access to `Array#transpose`. Both languages allow the `def` and `end` of the method body to be used as the `begin` and `end` of the `rescue` statement, but Crystal requires an Error instance rather than just an error class name.

### Problem Design

The object being required here is not very useful. Either this `compute` method is intended to be called on exactly two objects, in which case it should probably be implemented as `String#hamming_code`, or it should probably be some sort of mix-in extension (this is much more easily accomplished in Ruby). 
