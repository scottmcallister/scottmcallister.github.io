---
layout: post
title: "Thoughts on Redux"
date: 2016-10-25 07:03
image: '/assets/img/'
description: 'Some food for thought on the popular React library'
tags:
- react
- redux
- javascript
categories:
- Web Development
---

## Why Redux? ##

The real benefits of using Redux weren't incredibly obvious to me when I first
started building interfaces with it. I didn't see the point of having
to touch three separate files to send an API request, handle a button click,
or make a simple change to the DOM. I was used to the "simplicity" of plug and
play JS libraries like jQuery where most if not all of these tasks could be
handled by calling a single function or binding an event handler to the document.

Unfortunately, relying exclusively on libraries like jQuery for handling
asynchronous requests and DOM manipulation will only make life easier
up to a point. Once you start toggling visibility, creating and destroying
elements on the page, and changing CSS properties based on different user
interactions, trying to handle and test for all possible states becomes
increasingly difficult. This is the problem that Redux claims to solve, and
after using Redux for several months I can see how it makes handling state on
the front end a hell of a lot easier. After experiencing first hand the headache
of having to navigate through hundreds of lines of nested inline jQuery event
handler code to fix a bug or implement a new feature, I can safely say that in
many cases it's well worth the hastle of having to edit more than one file to make
a change.

To see if React and Redux makes sense for the application you're building, feel
free to check out <a href="https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367#.vpjibudwk" target="_blank">this article.</a>

## The Architecture ##

If you've spent any amount of time working with Redux, you've probably seen some
version of this diagram.

<img src="/assets/img/redux-flow.png" style="width: 80%; margin: auto;">

The diagram describes the flow of data fairly well. Components dispatch actions,
actions return an object for the reducer to handle, reducers take that object
and update the application state, and finally the store passes new props to a
component which in turn updates the view.

The first time I looked at this diagram, I assumed that this also
described how the directory structure of a Redux application should look. I
figured components should all be grouped in one directory, actions in their own
directory, reducers in another, and the store definition at the root of the
application. In fact if you look at the "real world" example on the
<a href="https://github.com/reactjs/redux/tree/master/examples/real-world" target="_blank">
Redux Github</a> project, it will look something like this:

<pre>
<code>
src/
│   store.js
│   root-reducer.js
│
└───actions/
│   │   thing-one.js
│   │   thing-two.js
│   │   thing-three.js
│
└───components/
│   │   thing-one.jsx
│   │   thing-two.jsx
│   │   thing-three.js
│
└───reducers/
    │   thing-one.js
    │   thing-two.js
    │   thing-three.js
</code>
</pre>

One issue that this sort of directory structure runs into down the road is that
the more actions and components you have, the harder this structure becomes to
navigate. Imagine this same pattern applied not to just a few components,
but to several hundred. Even if you were to create several
sub-directories in the actions, components, and reducers sections, you'll still
have to navigate to three completely separate parts of your application to make
any sort of state change.

## A Better Way ##

What I discovered after working on a pre-existing Redux application is that the
directory structure is by no means set in stone. Instead of grouping your files
based on what part of the data flow they are a part of, you can group them based
on what part of the UI they are associated with. What you end up then is a
directory structure that looks sort of like this:

<pre>
<code>
src/
│   root-reducer.js
│   store.js
│
└───scenes/
    │
    └───auth/
    │   │   actions.js
    │   │   index.js
    │   │   reducer.js
    │   │
    │   └───components/
    │       │   login.jsx
    │       │   forgot-password.jsx
    │       │   signup.jsx
    │
    └───dashboard/
        │   actions.js
        │   index.js
        │   reducer.js
        │
        └───components/
            │   section-one.jsx
            │   section-two.jsx
            │   section-three.jsx
</code>
</pre>

Not only does this structure make it easier to find any specific component
based on where it's used in the UI, but it also helps you locate the actions
and reducers associated with variables in your state tree. If you were to
have several root reducers in your app across multiple sub-sections, you could
follow pretty much the same path in your directory tree as you would follow in
your state tree

## Redux Devtools Extension ##

After working with Redux for quite some time I finally received one tip from a
co-worker that completely changed my workflow and helped me see why Redux is
so great; use the Redux Devtools Extension. If you're using Redux and you haven't already installed the
Redux Devtools browser extension, I can tell you that using it will be one
of the best things you can do to improve your Redux development workflow.

With this extension you can see a complete ordered history of every action
that's been dispatched in your application and the value of your application's
state tree after each action. If there is literally any problem with your Redux
data flow, this tool can help point you in the right direction to see why it
isn't working the way you want it to.

Let's say you've implemented an action and a reducer case that handles a button
click in one of your components. After writing and transpiling your code, you
try testing the change in your browser, but you don't see the change working.

Don't see the action listed after clicking? It's probably not being dispatched
in your component. Don't see a state change for an action that's been called?
The reducer probably isn't handling the action properly. Don't see the component
re-rendering on a state change? The state probably isn't mapped correctly to the
component's props in the `connect()` function.

This extension also illustrates why having a single store for your application
state is so useful. The ability to go back to any point your application's
history becomes trivial when all you need to do is change one JSON value.

## Final Thoughts ##

Despite adding some complexity to smaller scale applications, the conventions
that Redux uses can help manage state in apps that are larger in scale. Using
Redux Devtools and an alternative directory structure has helped me with
handling the complexity of Redux.
