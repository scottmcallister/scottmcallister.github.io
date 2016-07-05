---
layout: post
title: "Meteor Authentication With React Router"
date: 2016-07-05 07:03:35
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


The "target" div is where we will be rendering our app's React components. Lets try
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
will execute as soon as the DOM is ready. This function will call React's
`render()` function and render the component passed as the first parameter
inside our div. In this case the component being passed is a just a heading
that says "Test", but we'll be replacing this with something else very soon.

At this point we should try opening our app in a browser to make sure our code is
working correctly. Try running `meteor` inside the app's directory in a terminal
and opening `localhost:3000` in your browser. You should see the word "Test" if
everything runs smoothly.

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

We're including our components inside an "imports" directory in our project. Meteor
has a few [conventions][meteor-directory-structure] you should be aware of when building out your app's directory
structure. Scripts within certain directories in a Meteor application will be executed
on the client, the server, or both depending on which directory they are included in.

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

The routes file in our app has three routes; one for login, one for
signup, and an index route. The index route is nested within an AppContainer, which we will use to
make sure users are logged in before they can see the main page of our app.

We're also nesting our MainPage inside a MainContainer component to provide data
to our main page. Meteor provides a "[createContainer][create-container]" method to help us access
reactive data sources within our rendered views.

<div class="code-description">
  <p>imports/ui/containers/AppContainer.jsx</p>
</div>
<pre>
<code class="language-jsx">import React, { Component, PropTypes } from 'react';
import { browserHistory } from 'react-router'

export default class AppContainer extends Component {
  constructor(props){
    super(props);
    this.state = this.getMeteorData();
    this.logout = this.logout.bind(this);
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

  logout(e){
    e.preventDefault();
    Meteor.logout();
    browserHistory.push('/login');
  }

  render(){
    return (
      &lt;div>
        &lt;nav className="navbar navbar-default navbar-static-top">
          &lt;div className="container">
            &lt;div className="navbar-header">
              &lt;a className="navbar-brand" href="#">Auth App&lt;/a>
            &lt;/div>
            &lt;div className="navbar-collapse">
              &lt;ul className="nav navbar-nav navbar-right">
                &lt;li>
                  &lt;a href="#" onClick={this.logout}>Logout&lt;/a>
                &lt;/li>
              &lt;/ul>
            &lt;/div>
          &lt;/div>
        &lt;/nav>
        {this.props.children}
      &lt;/div>
    );
  }
}

</code>
</pre>

This container will check to see if there is a user logged into a session before
rendering. If no user session has been found, the user will be redirected to the
login page.

We've also included a navigation bar at the top of the container. In a real
application I would recommend abstracting this nav bar into its own
separate component. I've decided to include this navigation bar directly in the
render method to reduce the number of files we need for this tutorial.

<div class="code-description">
  <p>imports/ui/pages/LoginPage.jsx</p>
</div>
<pre>
<code class="language-jsx">import React, { Component, PropTypes } from 'react'
import { browserHistory, Link } from 'react-router'
import { createContainer } from 'meteor/react-meteor-data'

export default class LoginPage extends Component {
  constructor(props){
    super(props);
    this.state = {
      error: ''
    };
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(e){
    e.preventDefault();
    let email = document.getElementById('login-email').value;
    let password = document.getElementById('login-password').value;
    Meteor.loginWithPassword(email, password, (err) => {
      if(err){
        this.setState({
          error: err.reason
        });
      } else {
        browserHistory.push('/');
      }
    });
  }

  render(){
    const error = this.state.error;
    return (
      &lt;div className="modal show">
        &lt;div className="modal-dialog">
          &lt;div className="modal-content">
            &lt;div className="modal-header">
              &lt;h1 className="text-center">Login&lt;/h1>
            &lt;/div>
            &lt;div className="modal-body">
              { error.length > 0 ? &lt;div className="alert alert-danger fade in">{error}&lt;/div> :''}
              &lt;form id="login-form" className="form col-md-12 center-block" onSubmit={this.handleSubmit}>
                &lt;div className="form-group">
                  &lt;input type="email" id="login-email" className="form-control input-lg" placeholder="email"/>
                &lt;/div>
                &lt;div className="form-group">
                  &lt;input type="password" id="login-password" className="form-control input-lg" placeholder="password"/>
                &lt;/div>
                &lt;div className="form-group text-center">
                  &lt;input type="submit" id="login-button" className="btn btn-primary btn-lg btn-block" value="Login" />
                &lt;/div>
                &lt;div className="form-group text-center">
                  &lt;p className="text-center">Don't have an account? Register &lt;Link to="/signup">here&lt;/Link>&lt;/p>
                &lt;/div>
              &lt;/form>
            &lt;/div>
            &lt;div className="modal-footer" style=&#123;{borderTop: 0}}>&lt;/div>
          &lt;/div>
        &lt;/div>
      &lt;/div>
    );
  }
}
</code>
</pre>

Our login page is overriding the form's submit event and calling Meteor's
`loginWithPassword()` function to create a session. We're also showing  
authentication errors by modifying the component's state when there's an error
logging in.

<div class="code-description">
  <p>imports/ui/pages/SignupPage.jsx</p>
</div>
<pre>
<code class="language-jsx">import React, { Component, PropTypes } from 'react'
import { browserHistory, Link } from 'react-router'
import { Accounts } from 'meteor/accounts-base'

export default class SignupPage extends Component {
  constructor(props){
    super(props);
    this.state = {
      error: ''
    };
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(e){
    e.preventDefault();
    let name = document.getElementById("signup-name").value;
    let email = document.getElementById("signup-email").value;
    let password = document.getElementById("signup-password").value;
    this.setState({error: "test"});
    Accounts.createUser({email: email, username: name, password: password}, (err) => {
      if(err){
        this.setState({
          error: err.reason
        });
      } else {
        browserHistory.push('/login');
      }
    });
  }

  render(){
    const error = this.state.error;
    return (
      &lt;div className="modal show">
        &lt;div className="modal-dialog">
          &lt;div className="modal-content">
            &lt;div className="modal-header">
              &lt;h1 className="text-center">Sign up&lt;/h1>
            &lt;/div>
            &lt;div className="modal-body">
              { error.length > 0 ? &lt;div className="alert alert-danger fade in">{error}&lt;/div> :''}
              &lt;form id="login-form" className="form col-md-12 center-block" onSubmit={this.handleSubmit}>
                &lt;div className="form-group">
                  &lt;input type="text" id="signup-name" className="form-control input-lg" placeholder="name"/>
                &lt;/div>
                &lt;div className="form-group">
                  &lt;input type="email" id="signup-email" className="form-control input-lg" placeholder="email"/>
                &lt;/div>
                &lt;div className="form-group">
                  &lt;input type="password" id="signup-password" className="form-control input-lg" placeholder="password"/>
                &lt;/div>
                &lt;div className="form-group">
                  &lt;input type="submit" id="login-button" className="btn btn-lg btn-primary btn-block" value="Sign Up" />
                &lt;/div>
                &lt;div className="form-group">
                  &lt;p className="text-center">Already have an account? Login &lt;Link to="/login">here&lt;/Link>&lt;/p>
                &lt;/div>
              &lt;/form>
            &lt;/div>
            &lt;div className="modal-footer" style=&#123;{borderTop: 0}}>&lt;/div>
          &lt;/div>
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
app's database.

<div class="code-description">
  <p>imports/ui/pages/MainPage.jsx</p>
</div>
<pre>
<code class="language-jsx">import React, { Component, PropTypes } from 'react'
import { browserHistory, Link } from 'react-router'

export default class MainPage extends Component {
  constructor(props){
    super(props);
    this.state = {
      username: ''
    };
  }

  render(){
    let currentUser = this.props.currentUser;
    let userDataAvailable = (currentUser !== undefined);
    let loggedIn = (currentUser && userDataAvailable);
    return (
      &lt;div>
        &lt;div className="container">
          &lt;h1 className="text-center">{ loggedIn ? 'Welcome '+currentUser.username : '' }&lt;/h1>
        &lt;/div>
      &lt;/div>
    );
  }
}

MainPage.PropTypes = {
  username: React.PropTypes.string
}

</code>
</pre>

<div class="code-description">
  <p>imports/ui/containers/MainContainer.jsx</p>
</div>
<pre>
<code class="language-jsx">import { createContainer } from 'meteor/react-meteor-data';
import MainPage from '../pages/MainPage.jsx'

export default MainContainer = createContainer(({params}) => {
  const currentUser = Meteor.user();
  return {
    currentUser,
  };
}, MainPage);
</code>
</pre>

The main page is simply showing the user's name in a "welcome" message. Obviously
this isn't a realistic use case for a Meteor app, but it does show how you
can access reactive data sources in your React components by using the
`createContainer()` function.  

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

We should now have a fully functional Meteor app with user authentication. Try running
your app by using the `meteor` command within your app's directory in a terminal. Opening
`localhost:3000` in your browser should redirect you to the app's login page. If your
app is not working for whatever reason, the source code for this tutorial is available
[here][github-repo] for your reference.

As you can see, setting up user authentication with custom views is entirely possible
and fairly straight forward in Meteor when using React and React Router. While there
are modules like `accounts-ui` that you can use to add login and signup views to your app, building
out these views yourself will give you the freedom to fully customize what's being shown to your users.

[meteor-install]: https://www.meteor.com/install
[create-container]: https://guide.meteor.com/react.html#using-createContainer
[meteor-directory-structure]: https://guide.meteor.com/structure.html#example-app-structure
[github-repo]: https://github.com/scottmcallister/auth-app
