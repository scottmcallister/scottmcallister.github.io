---
layout: post
title: "Caching in Rails"
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

<h2>Fragment Caching</h2>

Just like the name sounds, fragment caching involves only caching parts of your page.
This is a great strategy for improving load times on pages with dynamic content. Since
fragment caching is a core feature of Rails, it can be used without having to rely on
any third party gems.

Now let's create a model, controller, route, and view that implements fragment caching.

<pre>
<code>$ rails g model Post title:string
$ rake db:migrate</code>
</pre>

<div class="code-description">
  <p>app/controllers/posts_controller.rb</p>
</div>
<pre>
<code class="language-ruby">class PostsController < ApplicationController
  def index
    @posts = Post.all
  end
end</code>
</pre>

<div class="code-description">
  <p>config/routes.rb</p>
</div>
<pre>
<code class="language-ruby">Rails.application.routes.draw do
  resources :posts, only: [:index]
  root "posts#index"
end</code></pre>

<div class="code-description">
  <p>views/posts/index.html.erb</p>
</div>
{% highlight erb %}
<h1>Posts</h1>

<% @posts.each do |post| %>
  <%= render post %>
<% end %>
{% endhighlight %}

<div class="code-description">
  <p>views/posts/_post.html.erb</p>
</div>
{% highlight erb %}
<h3>post.title</h3>
{% endhighlight %}

Now that we have our display logic ready to go, let's fill our database with
some data:

<div class="code-description">
  <p>db/seeds.rb</p>
</div>
<pre>
<code class="language-ruby">(1..10000).each do |i|
  Post.create!({title: "Post #{i}"})
end</code></pre>

<pre>
<code>$ rake db:seed</code>
</pre>

Try loading your app in a browser to see what kind of page load speeds we're
getting. The initial page load times I saw while testing were around 2000ms for
each request.

Let's see how this improves by adding fragment caching:

<div class="code-description">
  <p>views/posts/index.html.erb</p>
</div>
{% highlight erb %}
<% @posts.each do |post| %>
  <% cache post do %>
    <%= render post %>
  <% end %>
<% end %>
{% endhighlight %}

After adding in fragment caching, the initial page load in my test was
significantly slower at around 5000ms. This makes sense, since each of the 1000
fragments loaded on the page will need to create a new cache entry when they are
rendered. The subsequent requests however were loading MUCH faster at around 800ms.
That's less than half the time it took before we implemented fragment caching!
