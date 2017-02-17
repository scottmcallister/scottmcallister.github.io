---
layout: post
title: "Redux Without A Framework"
date: 2017-02-15 07:03:35
image: '/assets/img/'
description: 'How to use Redux without using a framework'
tags:
- javascript
- redux
categories:
- Web Development
---

Up until now, any Javascript development that I've worked on involving Redux
has also involved using React. In fact, most of the Redux examples that
I've seen on the web also use some sort of front end framework/library
(mostly React and Angular).

Over the past few days I've built a simple version of the game Tic Tac Toe using
Redux and plain ES6. In this post, I'm going to go through the steps I took to
create my game.


## Setting up ##

First, let's create a new directory to put our app in and install some NPM
packages.

<pre>
<code>$ mkdir tic-tac-toe; cd tic-tac-toe
$ npm init
$ npm install --save-dev webpack webpack-dev-server babel-core babel-loader babel-preset-es2015 path</code>
</pre>

At this point you should see a package.json file and a node_modules directory
inside your project folder. Next we'll add a webpack config file so we can
transpile our ES6 code to ES5 using webpack and babel.

<div class="code-description">
  <p>./webpack.config.js</p>
</div>
<pre>
<code class="language-javascript">var path = require('path');
var webpack = require('webpack');

module.exports = {
    entry: './js/index.js',
    output: {
        filename: './dist/bundle.js'
    },
    module: {
        loaders: [
            {
                loader: 'babel-loader',  /* eslint-disable */
                test: path.join(__dirname, 'js'), /* eslint-enable */
                query: {
                    presets: 'es2015'
                }
            }
        ]
    },
    plugins: [
        // Avoid publishing files when compilation fails
        new webpack.NoEmitOnErrorsPlugin()
    ],
    stats: {
        // Nice colored output
        colors: true
    },
    // Create Sourcemaps for the bundle
    devtool: 'source-map'
};</code>
</pre>

Basically this file is telling webpack to transpile any Javascript files in the
'js' folder from ES6 to ES5 and put the transpiled code into './dist/bundle.js'.
There's two comments I've included here that will disable linting warnings for
the '__dirname' variable. If you aren't using ESlint currently, I'd highly
recommend trying it out (even though it's not really required for this
tutorial).

Now let's add an HTML document and some JS code to our project.

<div class="code-description">
  <p>./index.html</p>
</div>
<pre>
<code class="language-html">&lt;head>
  &lt;title>Tic Tac Toe&lt;/title>
&lt;/head>

&lt;body>
&lt;script src="dist/bundle.js">&lt;/script>
&lt;/body></code>
</pre>

<div class="code-description">
  <p>./js/index.js</p>
</div>
<pre>
<code class="language-javascript">const name = 'world';
const message = `Hello ${name}!`;
console.log(message);</code>
</pre>

Now that we have
