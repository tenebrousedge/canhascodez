---
title: "Hashes in Ruby and Crystal"
date: 2019-04-09T23:35:35-07:00
summary: "There's more than one way to do it!"
featuredImage: "images/jon-tyson-478928-unsplash-minified.jpg"
imageCredit: <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@jontyson?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Jon Tyson"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Jon Tyson</span></a>
---

### Hashes in Ruby and Crystal

Ruby has a ton of different ways to write hashes, particularly ones involving strings or symbols. In addition to the normal explicit syntax, we have:
```ruby
BASES = Hash['C', 'G', 'G', 'C', 'T', 'A', 'A', 'U']

BASES = ['CGTA','GCAU'].map(&:chars).transpose.to_h

BASES = [%w{C G T A}, %w{G C A U}].transpose.to_h

BASES = %w{C G G C T A A U}.each_cons(2).to_h

BASES = Hash[?C, ?G, ?G, ?C, ?T, ?A, ?A, ?U]

BASES = %w{G C T A}.zip(%w{C G A U}).to_h

BASES = Hash[*%w{C G G C T A A U}]

BASES = Hash[*"CGGCTAAU".chars]
```
Crystal has somewhat fewer:
```crystal
BASES = ["CGTA", "GCAU"].map(&.chars).transpose.to_h

BASES = [%w{C G T A}, %w{G C A U}].transpose.to_h

BASES = %w{C G G C T A A U}.each_cons(2).to_h

BASES = Hash.zip(%w{G C T A}, %w{C G A U})

BASES = %w{G C T A}.zip(%w{C G A U}).to_h
```
Try to avoid using any of these in production code: there's not a hard line between "clever" and "obfuscated".
