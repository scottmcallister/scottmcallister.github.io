---
layout: post
title: "Caching in Rails"
date: 2016-07-20 07:03:35
image: '/assets/img/'
description: 'A quick look at how caching can improve performance in Ruby on Rails applications'
tags:
- ruby
- rails
categories:
- Web Development
---

Ever since Twitter publicly announced they had replaced their Ruby on Rails search
architecture with Scala back in 2011, the performance and scalability of Rails apps
has been challenged by the web development community. Although Twitter opted to
rewrite their entire server side architecture to solve their performance problems, sites
like Github and Airbnb still use Ruby on Rails and are able to serve content to
millions of users without unbearable latency. Writing performant web apps using
Ruby on Rails is clearly possible and caching is one of the many tools at your disposal
to maximize the performance of your Rails apps.
