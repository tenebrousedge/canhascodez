---
title: "Exercism № 3: RNA Transcription"
date: 2019-03-25T21:21:44-07:00
categories: ["programming", "exercism"]
tags: ["ruby", "crystal", "exercism"]
summary: "Taking a deep dive into string replacement methods."
featuredImage: 'images/dna-3539309_1920.jpg'
imageCredit: "<a href='https://pixabay.com/illustrations/dna-genetic-material-helix-proteins-3539309/'>Pixabay</a>"
---

## A Few of My Favorite Strings

Our third [exercism](https://exercism.io) problem is "RNA Transcription", which I'd like to say is a slightly better designed problem. Previous versions of this problem required returning `""` for bad input. Currently this is not required, which is good, because there is no way that could possibly be sane behavior. This also simplifies the code required quite a bit. 
There are a couple different approaches which bear mention. The first is a good example of what _not_ to do:
```ruby
class Complement
  def self.of_dna(str)
  rna = ''
  str.split(//).each do |base|
    case base
      when 'G'
        rna << 'C'
      when 'C'
        rna <<'G'
      when 'T'
        rna << 'A'
      when 'A'
        rna << 'U'
      else
        return ''
    end
  end
  rna
end
```
This code is gross. It's not that it's wasteful or that it doesn't work, but it operates at a very low level of abstraction, and no part can be easily be re-used. Programmers should work at the highest level of abstraction which is practical, and the most flexible solutions are generally obtained by encoding the logic of the problem into an `Enumerable` data structure. In no particular order:

  * If you're not passing a complex argument to `split`, don't use it. If you can, use `each_char` or `each_line`, and if you must, use `chars`. `split` is not bad *per se*, but most often there are better alternatives.
  * Declaring a container variable on a separate line, or using a line to return one, are code smells (and are almost always found together). I don't know that it's true that this is never necessary, but one should always strive to avoid this.
  * Do not use `return` inside a loop, if at all possible. We'll discuss some alternatives. It's okay to raise an exception, and it might be acceptable to stop evaluating a complex loop and return immediately, but sticking a return in the `else` of a `case` statement in a loop to return an non-useful non-error value is on the same list of "Bad Programmer Ideas" as "storing passwords in plain text".
  * Try to avoid mutating strings. Ruby is heading down the path of immutable strings, and Crystal is already there. There's usually a better alternative.
  * If possible, do not use `str` or `int` as variable names. Variable names should be meaningful.

It should be noted that because strings are not mutable in Crystal, the above code is not valid Crystal code. Other differences will be noted, but as always, most valid Ruby code is also valid Crystal code (modulo `&:` <-> `&.`)

### String#tr

Going a little bit up the abstraction ladder, we have an extremely concise solution, which is arguably the best by virtue of being simplest.
```ruby
def self.of_dna(rna_strand)
  return '' unless strand.match?(/\A[GCTA]+\z/) 
  rna_strand.tr('GCTA', 'CGAU')
end
```
This could be cleaned up a little with some constants:
```ruby
DNA_BASES = "GCTA"
RNA_BASES = "CGAU"
def self.of_dna(rna_strand)
  return '' unless strand.match?(/\A[#{DNA_BASES}]+\z/)
  rna_strand.tr(DNA_BASES, RNA_BASES)
end
```
The benefits of concision are eroded, and the data structures don't capture the association between the base pairs: that information must be inferred from the other code. Also, the string must be traversed twice, because `tr` will happily ignore any garbage data in the input strand. The question of whether to throw an error at all is debatable, but probably not decidable in context.

Anyway, let's stick these character sets in an associative data structure:
Ruby:
```ruby
DNA_TO_RNA = ["GCTA", "CGAU"]
"GATTACA".tr(*DNA_TO_RNA)
#=> "GUAAUGU"
```
Crystal:
```crystal
DNA_TO_RNA = {"GCTA", "CGAU"}
"GATTACA".tr(*DNA_TO_RNA)
#=> "GUAAUGU"
```
Hmm. Probably not what I meant. Note that the object used in the Crystal example is a [`Tuple`](https://crystal-lang.org/api/0.27.2/Tuple.html), not an `Array` or `Hash`. The splat `*` in Crystal is essentially only useful with tuples, which is somewhat limiting in cases where one might want to write:
```ruby
head, *rest = some_collection
```
However, having to use a range or slice in that situation isn't that bad:
```crystal
some_collection[0], some_collection[1..-1]
```
## `Hash`-based Solutions
Anyway, let's look at a real associative data structure, `Hash`:
```ruby
DNA_TO_RNA = {
  'G' => 'C',
  'C' => 'G',
  'T' => 'A',
  'A' => 'U'
}
"CATCATCAT".tr(DNA_TO_RNA.keys.join, DNA_TO_RNA.values.join)
#=> "GUAGUAGUA"
```
Having to use `keys` and `values` is doable, but a little awkward. I think we can conclude that `tr` is useful for when the replacements are string values, not collections. That may or may not be a useful model of reality, but reality doesn't necessarily have much to do with code at the best of times, and here we're just exploring alternatives. A more effective strategy is to use `gsub`:
Ruby:
```ruby
DNA_TO_RNA = {
  'G' => 'C',
  'C' => 'G',
  'T' => 'A',
  'A' => 'U'
}
"TAGACAT".gsub(/./, DNA_TO_RNA)
#=> "AUCUGUA"
```
Crystal:
```crystal
DNA_TO_RNA = {
  'G' => 'C',
  'C' => 'G',
  'T' => 'A',
  'A' => 'U'
}
"CATACT".gsub(DNA_TO_RNA)
#=> "GUAUGA"
```
Crystal supports passing the hash to `gsub` directly. Ruby and Crystal both support passing both a regex and a hash, as well as a number of other invocations: `gsub` is a complex beast. 

## Loops and Broken Things

The current version of this problem does not involve any error handling, which is probably good, because the previous version of the problem had people return an empty string for bad input, and there's no real possibility that could have been correct behavior. It's probably usually best to do sanity checks before trying to process your data, but sometimes, you just have to "bail out" from a loop.

The obvious way to return from the middle of a loop would be to use the `return` keyword.
```ruby
"CATACGG".each_char.map { |base| DNA_TO_RNA[base] || return '' }.join
```
Forget that this is possible. There is never a time when this is the preferred method of exiting a loop, and it's highly likely to trip up the unwary reader of your code. Among other things, it's not necessarily obvious that the `return` exits the entire method, not just the loop. What are some better options? *If you want to exit a loop and return a non-error value, use `break`*:
```ruby
"GAGACAT".each_char.map { |base| DNA_TO_RNA[base] || break [] }.join
```
If this is an error condition, you could `raise` an appropriate exception. Exceptions are signals which "bubble up" until they are either caught, or the program execution halts. You should probably try to have a really good reason for doing that in the middle of a loop, and prefer to place a "guard clause" before your main loop to catch that condition. However, failing that, it's probably a good idea to encapsulate that logic in the object you're using. In Ruby, you can do:
```ruby
DNA_TO_RNA = Hash.new { raise InvalidDnaError }.merge({
  'G' => 'C',
  'C' => 'G',
  'T' => 'A',
  'A' => 'U'
})
```
If it's not clear there, we're passing a block to `Hash.new`, then merging in a Hash literal. Ruby can also perform the operations in reverse order:
```ruby
DNA_TO_RNA = {
  'G' => 'C',
  'C' => 'G',
  'T' => 'A',
  'A' => 'U'
}.tap { |hash| hash.default_block = -> { raise InvalidDnaError } }
```
In Crystal, at the time of this writing, only `merge!` preserves any default block on the receiver. Also, default values are implemented as default blocks that return that value. Also, Crystal lacks any way to set a default block (or default value) other than with `new`. This makes it somewhat more difficult to work with hashes with default values or blocks. I opened [issue 7683](https://github.com/crystal-lang/crystal/issues/7683) for this, and will be working to resolve this. 
