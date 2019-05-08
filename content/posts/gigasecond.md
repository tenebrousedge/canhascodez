---
title: "Exercism № 2: Gigasecond"
date: 2019-03-25T21:04:43-07:00
draft: true
categories: ["programming", "exercism"]
tags: ["crystal", "ruby", "exercism"]
summary: "We continue exploring Exercism problems in Ruby and Crystal with some simple time-related methods."
featuredImage: "images/aron-visuals-322314-unsplash-minified.jpg"
imageCredit: '<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@aronvisuals?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Aron Visuals"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Aron Visuals</span></a>'
---

## Somewhat More Than 525,600 Minutes

Another [Exercism](https://exercism.io/) exercise. This one is too simple to write much about.
Ruby:
```ruby
module Gigasecond
  GIGASECOND = IE9

def self.from(time)
    time + GIGASECOND
  end
end
```
Crystal:
```crystal
module Gigasecond
  GIGASECOND = 1_000_000_000.seconds

  def self.from(time : Time) : Time
    time + GIGASECOND
  end
end
```
Crystal's numeric objects have some nice convenience methods for dealing with time. You *can* create a `Time` object manually, but there really shouldn't be a need to do so. The type annotations are not required, but could be considered a good idea. Storing the gigasecond value as a constant is good practice.

That's about it. On to the next problem!

