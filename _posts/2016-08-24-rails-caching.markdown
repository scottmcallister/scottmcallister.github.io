---
layout: post
title: "Caching in Rails"
date: 2016-08-24 07:03
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
  <p>controllers/posts_controller.rb</p>
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
<code class="language-ruby">(1..1000).each do |i|
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
That's less than half the time it took before!

<h2>Russian Doll Caching</h2>

The next form of caching we're going to explore is called "Russian Doll caching".
This form of caching involves caching fragments that are contained inside other
cached fragments.

Let's say your app has cached fragments inside each of your post fragments to
represent comments for that post. If any of these comments are updated, the post
fragment will still show a cached version of its comments since the post record
hasn't been updated.

We can scaffold out comments for our posts and insert a comment into the database.

<pre>
<code>$ rails g scaffold comment message:string post:belongs_to
$ rake db:migrate
$ rails db

sqlite> insert into comments(message, post_id, created_at, updated_at)
        values("first comment", 1, 'now', 'now');
sqlite> .exit</code>
</pre>

Now let's update our post model and our views to show each post's comments in our
app.

<div class="code-description">
  <p>models/post.rb</p>
</div>
<pre>
<code class="language-ruby">
class Post < ApplicationRecord
  has_many :comment, dependent: :destroy

  def comments
    all_comments = Comment.where(post_id: self.id)
    return all_comments
  end
end
</code>
</pre>

<div class="code-description">
  <p>views/comments/_comment.html.erb</p>
</div>
{% highlight erb %}
<p><%= comment.message %></p>
{% endhighlight %}

<div class="code-description">
  <p>views/_post.html.erb</p>
</div>
{% highlight erb %}
<h3><%= post.title %></h3>
<% post.comments.each do |comment| %>
  <%= render 'comments/comment', :comment => comment %>
<% end %>
{% endhighlight %}

If we take a look at our app now, we can see the comment that was inserted just
below the first post listing. Without restarting the server let's try inserting
another comment for the same post and reload our page. Try creating another comment
by heading to `/comments/new` and setting the post value to 1.

As you can see, the second comment isn't shown. If we restart the app to clear the
cache the second post should be displayed. To implement proper caching for
our comments, the first step is to cache the inner comment fragments:

<div class="code-description">
  <p>views/_post.html.erb</p>
</div>
{% highlight erb %}
<h3><%= post.title %></h3>
<% post.comments.each do |comment| %>
  <% cache comment do %>
    <%= render 'comments/comment', :comment => comment %>
  <% end %>
<% end %>
{% endhighlight %}

The second step is to use the `touch` method in the comment model.

<div class="code-description">
  <p>models/comment.rb</p>
</div>
<pre>
<code class="language-ruby">
class Comment < ApplicationRecord
  belongs_to :post, touch: true
end
</code>
</pre>

By setting touch to true, any action which updates a comment will also update the
post it belongs to. This will expire the post fragment's cache and prevent stale data
from being shown. Also, any comment fragments within that post that haven't been
updated will still be cached.

<h2>Wrapping Up</h2>

In this post we've explored two caching strategies you can use to improve the
performance of your Rails apps. There are a few other caching strategies available
that can be used to improve performance even further. Some of these strategies
have been removed from the core Rails framework in recent versions and placed in
separate gems. I'd suggest taking a look at the Ruby on Rails <a href="http://guides.rubyonrails.org/caching_with_rails.html" target="_blank">documentation</a>
for more information.

I think these strategies are a great first step towards maximizing the performance
of your Rails applications. Hopefully you'll find the examples in this post useful
as you implement caching for your own applications.
