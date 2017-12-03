---
title: 网页内容解析——Ruby实现
tags:
  - nokogiri
  - ruby
id: 599
categories:
  - 学习
date: 2014-12-02 15:52:42
---

本文继上篇文章"网页内容解析——Python实现"提供Ruby版本的实现方式

<!--more-->

Ruby中也有类似于jQuery解析HTML的方式的库——[nokogiri](http://www.nokogiri.org/)，还是上一篇的那个例子，实现代码也非常少，如下

```ruby
require 'rubygems'
require 'nokogiri'
require 'open-uri'

page = Nokogiri::HTML(open("http://item.jd.com/1164932931.html"))
page.css('div.breadcrumb a').each do |el|
  puts el.text
end
```

嗯，非常不错，已经帮我处理了编解码问题！！！

以下提供一些nokogiri的学习链接
1.[Parsing HTML with Nokogiri](http://ruby.bastardsbook.com/chapters/html-parsing/)
2.[RubyQuery - a simple way to parse HTML using jQuery style css query](http://ignition.hk/blog/2011/02/12/rubyquery/)