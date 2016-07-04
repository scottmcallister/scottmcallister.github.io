---
layout: post
title: "Custom Authentication in Meteor"
date: 2016-06-30 09:54:35
image: '/assets/img/'
description: 'A tutorial for writing custom authentication in a Meteor project using React Router'
tags:
- javascript
- react
- meteor
categories:
- Web Development
---

In this tutorial we will be writing a very simple Meteor app with custom built login
and signup pages. There will be one page that is password protected and will require
the user to be signed in. Our front end code will be written in React and we'll
use React Router for routing between views.

First, make sure that you have [Meteor installed][meteor-install] on your machine.
You'll also need Node JS and NPM for running the app and downloading the necessary
packages we'll be using. After you have everything installed, lets start our project.

<pre>
<code>$ meteor create auth-app
$ cd auth-app
$ meteor npm install --save react react-dom react-router react-addons-pure-render-mixin
$ meteor add accounts-password react-meteor-data twbs:bootstrap</code>
</pre>

After just a few commands we've created a Meteor project and installed the
necessary packages we'll need to build our app.

Replace the contents of `client/main.html` with this:

<div class="code-description">
  <p>client/main.html</p>
</div>
<pre>
<code class="language-html">&lt;head>
  &lt;title>Auth App&lt;/title>
&lt;/head>

&lt;body>
  &lt;div id="target">&lt;/div>
&lt;/body></code>
</pre>


The "target" div is where we will be rendering our app's React content. Lets try
rendering a component right now. Rename `client/main.js` to `client/main.jsx` and
replace all of the file's code with this:

<div class="code-description">
  <p>client/main.jsx</p>
</div>
<pre>
<code class="language-jsx">import React from 'react'
import { Meteor } from 'meteor/meteor'
import { render } from 'react-dom'

Meteor.startup(() => {
  render(&lt;h1>Test&lt;/h1>, document.getElementById('target'));
});</code>
</pre>

Here we're passing an anonymous function to Meteor's `startup()` function which
will execute as soon as the DOM is ready. This function will call react's
`render()` function and render the component passed as the first parameter
inside our div. In this case the component being passed is a just a heading
that says "Test", but we'll be replacing this with something else very soon.

At this point we should try opening our app in a browser to make sure our code is
working correctly. Try running `meteor` in the console and opening `localhost:3000`
in your browser. You should see the word "Test" if everything runs smoothly.

Now that we have components rendering properly, it's time to set up routing in
our app. Let's add some components and a route file to the project:

<div class="code-description">
  <p>new files</p>
</div>
<pre>
<code class="language-php">imports/startup/client/routes.jsx               # react router configuration
imports/ui/containers/AppContainer.jsx          # password protected container
imports/ui/containers/MainContainer.jsx         # a container to
imports/ui/pages/LoginPage.jsx                  # login page
imports/ui/pages/SignupPage.jsx                 # signup page
imports/ui/pages/MainPage.jsx                   # password protected page
</code>
</pre>

<div class="code-description">
  <p>imports/startup/client/routes.jsx</p>
</div>
<pre>
<code class="language-jsx">import React from 'react'
import { Router, Route, browserHistory, IndexRoute } from 'react-router'

// containers
import AppContainer from '../../ui/containers/AppContainer.jsx'
import MainContainer from '../../ui/containers/MainContainer.jsx'

// pages
import SignupPage from '../../ui/pages/SignupPage.jsx'
import LoginPage from '../../ui/pages/LoginPage.jsx'

export const renderRoutes = () => (
  <Router history={browserHistory}>
    <Route path="login" component={LoginPage}/>
    <Route path="signup" component={SignupPage}/>
    <Route path="/" component={AppContainer}>
      <IndexRoute component={MainContainer}/>
    &lt;/Route>
  &lt;/Router>
);

</code>
</pre>

The routes file has three routes; an index route, one for login, and one for
signup. The index route is nested within an AppContainer, which we will use to
make sure users are logged in before they can see the main page of our app.

We're also nesting our MainPage inside a MainContainer component to provide data
to our main page. Meteor provides a "createContainer" method to help us access
reactive data sources within our rendered views. For more information about how
this works you can check out Meteor's [documentation][create-container].

<div class="code-description">
  <p>imports/ui/containers/AppContainer.jsx</p>
</div>
<pre>
<code class="language-jsx">import React, { Component, PropTypes } from 'react'
import { createContainer } from 'meteor/react-meteor-data'
import { browserHistory } from 'react-router'

class AppContainer extends Component {

  constructor(props){
    super(props);
    this.state = this.getMeteorData();
  }

  getMeteorData(){
    return { isAuthenticated: Meteor.userId() !== null };
  }

  componentWillMount(){
    if (!this.state.isAuthenticated) {
      browserHistory.push('/login');
    }
  }

  componentDidUpdate(prevProps, prevState){
    if (!this.state.isAuthenticated) {
      browserHistory.push('/login');
    }
  }

  render(){
    return (
      &lt;div>
        {this.props.children}
      &lt;/div>
    );
  }
}

DashboardContainer.propTypes = {
  user: PropTypes.object
};

export default createContainer(() => {
  return {
    user: Meteor.user()
  };
}, AppContainer);
</code>
</pre>

This container will check to see if there is a user logged into a session before
rendering. If no user session has been found, the user will be redirected to the
login page.

<div class="code-description">
  <p>imports/ui/pages/LoginPage.jsx</p>
</div>
<pre>
<code class="language-jsx">import React, { Component, PropTypes } from 'react'
import { browserHistory } from 'react-router'
import { createContainer } from 'meteor/react-meteor-data'

export default class LoginPage extends Component {
  constructor(props){
    super(props);
    this.handleSubmit.bind(this);
  }

  handleSubmit(e){
    e.preventDefault();
    let email = document.getElementById('login-email').value;
    let password = document.getElementById('login-password').value;
    Meteor.loginWithPassword(email, password, (err) => {
      if(err){
        console.log(err);
      } else {
        browserHistory.push('/');
      }
    });
  }

  render(){
    return (
      &lt;div className="container">
        &lt;div className="row col-md-6 col-md-offset-3">
          &lt;form id="login-form" className="form-signin" onSubmit={this.handleSubmit}>
            &lt;h1 className="form-signin-heading text-center">Login&lt;/h1>
            &lt;input type="email" id="login-email" className="form-control" placeholder="email"/>
            &lt;input type="password" id="login-password" className="form-control" placeholder="password"/>
            &lt;input type="submit" id="login-button" className="btn btn-lg btn-primary btn-block" value="Login" />
          &lt;/form>
        &lt;/div>
      &lt;/div>
    );
  }
}
</code>
</pre>

Our login page component is overriding the form's submit event and calling Meteor's
`loginWithPassword()` function to create a session.

<div class="code-description">
  <p>imports/ui/pages/SignupPage.jsx</p>
</div>
<pre>
<code class="language-jsx">import React, { Component, PropTypes } from 'react'
import { browserHistory } from 'react-router'
import { Accounts } from 'meteor/accounts-base'

export default class SignupPage extends Component {
  constructor(props){
    super(props);
    this.handleSubmit.bind(this);
  }

  handleSubmit(e){
    e.preventDefault();
    let name = document.getElementById("signup-name").value;
    let email = document.getElementById("signup-email").value;
    let password = document.getElementById("signup-password").value;
    Accounts.createUser({email: email, name: name, password: password}, (err) => {
      if(err){
        console.log(err);
      } else {
        browserHistory.push('/login');
      }
    });
  }

  render(){
    return (
      &lt;div className="container">
        &lt;div className="row col-md-6 col-md-offset-3">
          &lt;form id="login-form" className="form-signin" onSubmit={this.handleSubmit}>
            &lt;h1 className="form-signin-heading text-center">Sign Up&lt;/h1>
            &lt;input type="text" id="signup-name" className="form-control" placeholder="name"/>
            &lt;input type="email" id="signup-email" className="form-control" placeholder="email"/>
            &lt;input type="password" id="signup-password" className="form-control" placeholder="password"/>
            &lt;input type="submit" id="login-button" className="btn btn-lg btn-primary btn-block" value="Sign Up" />
          &lt;/form>
        &lt;/div>
      &lt;/div>
    );
  }
}
</code>
</pre>

Our signup page component is very similar to our login page, only instead of calling
`loginWithPassword()`, we'll be creating a user with the accounts module's
`createUser()` function. This will insert a new user account document into our
app's Mongo database.

<div class="code-description">
  <p>imports/ui/pages/MainPage.jsx</p>
</div>
<pre>
<code class="language-jsx">import React, { Component } from 'react'

export default class MainPage extends Component {
  render(){
    let user = 'user';
    return (
      &lt;div>
      &lt;h1>Welcome &#123;{ user }}&lt;/h1>
      &lt;/div>
    );
  }
}
</code>
</pre>

The main page is simply showing the user's name and allowing the user to log out
by clicking a link in the nav bar. Obviously this isn't a realistic use case for
a Meteor app, but it illustrates how you can ensure most of your application is
password protected.

The last change you will need to make to set up routing in our application will
be in `client/main.jsx`. Instead of rendering the placeholder "test" component,
we will be including the `renderRoutes()` function from our routes file and
rendering that in our target div.

<div class="code-description">
  <p>client/main.jsx</p>
</div>
<pre>
<code class="language-jsx">import React from 'react';
import { Meteor } from 'meteor/meteor';
import { render } from 'react-dom';

// add render routes function
import { renderRoutes } from '../imports/startup/client/routes.jsx'

// render routes after DOM has loaded
Meteor.startup(() => {
  render(renderRoutes(), document.getElementById('target'));
});</code>
</pre>

[meteor-install]: https://www.meteor.com/install
[create-container]: https://guide.meteor.com/react.html#using-createContainer
