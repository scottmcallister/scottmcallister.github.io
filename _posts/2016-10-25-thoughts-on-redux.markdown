---
layout: post
title: "Thoughts on Redux"
date: 2016-10-25 07:03
image: '/assets/img/'
description: 'Some food for thought on the popular React library'
tags:
- React
- Redux
- Javascript
categories:
- Web Development
---

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
many cases it's well worth the hastle of having to edit multiple files to make
a change. 
