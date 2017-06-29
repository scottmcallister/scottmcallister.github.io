---
layout: post
title: "Using Templates With Spring Boot"
date: 2017-05-30 12:00:00
image: '/assets/img/'
description: 'A tutorial for displaying dynamic content with Spring Boot and Thymeleaf templates'
tags:
- java
- spring boot
categories:
- Web Development
---

In this tutorial we'll be building a very simple micro blog CRUD app. Rather than building a single page app for the UI that connects to a RESTful back end, we'll be generating dynamic views with the Thymeleaf template library. I'll be using Maven for dependency management in this tutorial, but feel free to use Gradle instead if that's your preference. 

To start off we'll need a new Spring Boot project with the following dependencies:

- `web`
- `thymeleaf`
- `data-jpa`
- `h2`
- `devtools`

If your IDE doesn't support creating new Spring Boot projects, you can use [Spring Initializr](https://start.spring.io/) to bootstrap a new project with these dependencies. Personally I find bootstrapping projects on a web interface really weird and prefer to use the [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started-installing-spring-boot.html) instead:

<pre>
<code>$ spring init --build=maven --dependencies=web,devtools,thymeleaf,data-jpa,h2 micro-blog-spring-boot
</code>
</pre>


