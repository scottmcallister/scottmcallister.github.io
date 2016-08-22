---
layout: post
title: "Caching in Rails 5"
date: 2016-08-22 07:03
image: '/assets/img/'
description: 'A quick look at how caching can improve the performance of Ruby on Rails applications'
tags:
- ruby
- rails
categories:
- Web Development
---

Caching can go a long way towards improving the performance of any web app, and Rails
apps are no exception. Ever since Twitter publicly announced they had replaced their
Ruby on Rails search architecture with Scala back in 2011, the performance and
scalability of Rails has been challenged by the web development community. Even
though Twitter opted to rewrite their server side architecture to solve their
performance problems, sites like Github, Shopify, and Airbnb still use Rails and
are able to serve content to millions of users with fast page load times. Writing
performant web apps using Ruby on Rails is clearly possible and caching is one
of the many tools at your disposal to maximize the performance of your Rails applications.

<h2>App Setup</h2>

By default, Rails applications will only enable caching in a production environment.
To create a Rails app with caching enabled, run the following commands:

<pre>
<code>$ rails new cache-app
$ cd cache-app
$ rake dev:cache</code>
</pre>
