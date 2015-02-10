+++
date = "2015-02-06T16:31:34-06:00"
title = "Getting Started"

+++

# On Information Technology

> Humans have been storing, retrieving, manipulating and communicating information since the Sumerians in Mesopotamia developed writing in about 3000 BC.[^1]

[^1]: [Wikipedia: Information Technology](http://en.wikipedia.org/wiki/Information_technology)

## First post
This is a "first post" on the new blog here - you'll see some older content moved over from another blog. I looked at [nanoc](http://nanoc.ws/) and [ruhoh](http://ruhoh.com/) as well as [Hugo](http://gohugo.io/) but Hugo seems to be a keeper. All of these tools are [static site generators](https://staticsitegenerators.net/) which meet my needs but I'd rather get started with something modern and easy. Another couple I'd like to look at are [middleman](https://middlemanapp.com/) and [hakyll](http://jaspervdj.be/hakyll/). At some point I'll develop some additional requirements but now my needs are simple: *write stuff now*.

My goals:

1. I like syntax colorizing for code. In fact this is an essential requirement. Fortunately this is [easy now](https://highlightjs.org/download/).

2. I like the idea of having some control over [themes](http://gohugo.io/themes/usage/) but also something that just works out of the box.

1. I'm familiar with markdown and its variants and it's well supported in RubyMine and other tools I use. It looks like Hugo uses [github flavored markdown](https://help.github.com/articles/github-flavored-markdown/) which is well documented and easy to use with code snippets.

Code highlighting is as easy as copying and pasting the code into the file and preceeding it with three backticks and a language type.

```ruby
    def setup
        @conn = Bunny.new("amqp://#{@opts[:server]}:5672")
        @conn.start

        ch = @conn.create_channel
        x = ch.topic("logs", :durable => true, :auto_delete => true)
        @q = ch.queue("", :exclusive => true)
        @q.bind(x, :routing_key => @opts[:key])
    end
```