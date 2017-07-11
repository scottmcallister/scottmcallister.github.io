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

In this tutorial we'll be building a very simple mini blog app. Rather than building a single page app for the UI that sends requests to a RESTful back end, we'll be generating dynamic views on the server with Thymeleaf templates. I'll be using Maven for dependency management in this tutorial, but feel free to use Gradle instead if that's your preference. 

To start off we'll need a new Spring Boot project with the following dependencies:

- `web`
- `thymeleaf`
- `data-jpa`
- `h2`
- `devtools`

If your IDE doesn't support creating new Spring Boot projects, you can use [Spring Initializr](https://start.spring.io/) to bootstrap a new project with these dependencies. Personally I find bootstrapping projects on a web interface a little weird and prefer to use the [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started-installing-spring-boot.html) instead:

<pre>
<code>$ spring init --build=maven --dependencies=web,devtools,thymeleaf,data-jpa,h2 micro-blog-spring-boot
</code>
</pre>

Now that we have a starting point for our project, let's write our first route. Add a "web" directory to your main package and put a file in there named `EntryController.java`. 

<div class="code-description">
    <p>src/main/java/com/example/microblogspringboot/web/EntryController.java</p>
</div>
<pre>
<code class="language-java">package com.example.microblogspringboot.web;

import com.example.microblogspringboot.repository.EntryRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class EntryController {
    @RequestMapping("/")
    public String home(Model model){
        model.addAttribute("title", "Hello World!");
        return "home";
    }
}
</code>
</pre>
The `@Controller` annotation sets our `EntryController` class as a Web MVC controller, and the `@RequestMapping` annotation will map all server requests for the home page to run this method. In Spring applications, controllers are responsible for preparing a model object with data and passing that model to a view to be rendered. By adding a "title" attribute to the model object in our method, our Thymeleaf template will be able to access the title variable and print its value on the page. 

The string value returned by this method is the name of the template file that will be used. The template name is simply the filename without the `.html` extension. Spring templates are typically stored in the `resources` directory in a folder called `templates`. Let's add a template file to our project. 

<div class="code-description">
    <p>src/main/resources/templates/home.html</p>
</div>
<pre>
<code class="language-html">&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
&lt;head>
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0" />
    &lt;title>Java Blog&lt;/title>
&lt;/head>
&lt;body>
    &lt;h1 th:text="${title}">&lt;/h1>
&lt;/body>
&lt;/html>
</code>
</pre>
The `th:text` attribute for our `h1` tag is injecting the title attribute we passed to our model. Also notice that the `meta` tag in our page head is ending with a `/>` as opposed to a `>`. Page elements that don't use a forward slash in HTML to close tags like `input` and `meta` will throw a syntax error in Thymeleaf if they are not closed with a forward slash before the ending caret.

Now we can test out our application to see if "Hello World!" was passed to our template properly. If you're using Maven, you can run your Spring Boot app with the following command:
<pre>
<code>$ mvn spring-boot:run
</code>
</pre>
For Gradle projects, you can start your app by using this command instead. 
<pre>
<code>$ gradle bootRun
</code>
</pre>
These commands can also be used to set up run configurations for your app inside whatever IDE you're using for Java development. After your server is up and running, try opening `localhost:8080` inside your browser to see "Hello World!" printed out. 

Now that we have an index route, we can start setting up the blog entry data for our app. Let's create a "domain" directory and put our new `Entry` model class inside it.

<div class="code-description">
    <p>src/main/java/com/example/microblogspringboot/domain/EntryController.java</p>
</div>
<pre>
<code class="language-java">package com.example.microblogspringboot.domain;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Entry {
    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    private Long id;
    private String title;
    private String content;

    protected Entry() {}

    public Entry(String title, String content) {
        this.title = title;
        this.content = content;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}
</code>
</pre>
As you can see, we've only set up three instance variables for this class: `id`, `title`, and `content`. The annotations above `id` indicate that this field is the primary key of our table and that the value of this field will not need to be set directly when creating new records. 

By using the `@Entity` annotation, Spring will automatically create a table for our class within our H2 database. Since H2 is an in-memory database, data within this table will be cleared each time we reboot our application. Obviously this isn't ideal for a real world application, but for the sake of holding temporary data for us to display in our templates it'll do the trick. 

To load blog entries into our database when the app starts up, we can place a SQL script in the resources directory named `data.sql`. 

<div class="code-description">
    <p>src/main/resources/data.sql</p>
</div>
<pre>
<code class="language-sql">INSERT INTO entry (title, content) VALUES
  ('First Post', 'I do not like green eggs and ham'),
  ('New Post', 'I do not like them Sam I am');
</code>
</pre>
We now have a model to create our entry table with and a SQL script to put records in that table. To actually query this table and return these records, we'll be using a repository class. This will act as a collection class of sorts we can use to get the blog entry records we need. 

<div class="code-description">
    <p>src/main/java/com/example/microblogspringboot/repository/EntryRepository.java</p>
</div>
<pre>
<code class="language-java">package com.example.microblogspringboot.repository;

import com.example.microblogspringboot.domain.Entry;
import org.springframework.data.repository.CrudRepository;
import java.util.List;

public interface EntryRepository extends CrudRepository&lt;Entry, Long> {
    List&lt;Entry> findAll();
}
</code>
</pre>
Since we're extending this class from Spring's `CrudRepository`, we'll be inheriting a bunch of methods for interacting with the entries stored in our database. The only method signiture we'll be overrriding will be the `findAll()` method. 

Now that we have some blog entries to show and a way to read them from our database, let's update the controller and template files we made earlier. 

<div class="code-description">
    <p>src/main/java/com/example/microblogspringboot/web/EntryController.java</p>
</div>
<pre>
<code class="language-java">package com.example.microblogspringboot.web;

import com.example.microblogspringboot.domain.Entry;
import com.example.microblogspringboot.repository.EntryRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import java.util.List;

@Controller
public class EntryController {
    @Autowired
    EntryRepository entryRepository;

    @RequestMapping("/")
    public String home(Model model){
        List&lt;Entry> allEntries = entryRepository.findAll();
        model.addAttribute("entries", allEntries);
        return "home";
    }
}
</code>
</pre>

<div class="code-description">
    <p>src/main/resources/templates/home.html</p>
</div>
<pre>
<code class="language-html">&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
&lt;head>
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0" />
    &lt;title>Java Blog&lt;/title>
&lt;/head>
&lt;body>
    &lt;h1>Blog Entries&lt;/h1>
    &lt;div th:each="entry : *{entries}">
        &lt;h2 th:text="${entry.title}">&lt;/h2>
        &lt;p th:text="${entry.content}">&lt;/p>
    &lt;/div>
&lt;/body>
&lt;/html>
</code>
</pre>
The changes we made to our controller will autowire an instance of our repository class that we can use to fetch all blog entries from our database and pass them to the view. 

In the template, we're using `th:each` to iterate through our list of blog entries and create a `div` element for each entry. In each div, we're placting the title of that entry inside an `h2` tag and the content inside a paragraph tag. If you open up the app now you should see titles and paragraphs for each blog entry. 

Now that we have our initial blog entries listed out, lets build a way to create new blog entries. We'll be using the same template for both creating and editing blog entries since both requests will be passing the same information. 

<div class="code-description">
    <p>src/main/resources/templates/entry.html</p>
</div>
<pre>
<code class="language-html">&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
&lt;head>
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0" />
    &lt;title th:text="${pageTitle}">&lt;/title>
&lt;/head>
&lt;body>
    &lt;h1 th:text="${pageTitle}">&lt;/h1>
    &lt;form method="post" th:action="${givenAction}">
        &lt;div>
            &lt;input name="title" type="text" placeholder="title" th:value="${givenTitle}" />
        &lt;/div>
        &lt;div>
            &lt;textarea name="content" type="text" placeholder="content" th:text="${givenContent}">&lt;/textarea>
        &lt;/div>
        &lt;div>
            &lt;button type="submit">Post&lt;/button>
        &lt;/div>
    &lt;/form>
&lt;/body>
&lt;/html>
</code>
</pre>
As you can see, there's quite a few attributes that we'll be passing to this view. The `pageTitle` will be used for both the `h1` tag and for the title tag in the header. Since we're reusing this template for both editing and creating entries, this title will be different depending on what sort of request has been made. The `givenTitle` and `givenContent` attributes will be empty strings for new entries, but editing existing entries will use the current values instead. The `th:action` attribute in our form will use the `givenAction` attribute to send requests to different endpoints depending on whether we're creating or editing entries. 

Lets write our new controller routes for creating a new blog entry.

<div class="code-description">
    <p>src/main/java/com/example/microblogspringboot/web/EntryController.java</p>
</div>
<pre>
<code class="language-java">package com.example.microblogspringboot.web;

// imports...
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class EntryController {
    
    // repository and home route...

    @RequestMapping(value = "/entry", method = RequestMethod.GET)
    public String newEntry(Model model) {
        model.addAttribute("pageTitle", "New Entry");
        model.addAttribute("givenAction", "/entry");
        model.addAttribute("givenTitle", "");
        model.addAttribute("givenContent", "");
        return "entry";
    }

    @RequestMapping(value = "/entry", method = RequestMethod.POST)
    public String addEntry(@RequestParam String title, @RequestParam String content) {
        Entry newEntry = new Entry(title, content);
        entryRepository.save(newEntry);
        return "redirect:/";
    }
}
</code>
</pre>
The `newEntry` method is pretty similar to our home route. We're basically accepting a GET request for `/entry` with no parameters, adding a few attributes to our template model, and loading the a template. 

Our `addEntry` method accepts a POST request for `/entry` with two parameters in the post body: `title` and `content`. It then takes those parameters and creates a new `Entry` object, then uses the `save` method that our repository class inherited from Spring's `CrudRepository` to add a new record to our entry table. Instead of returning a new template to render, we're redirecting the user back to the home page.

The last thing we'll need to add to start creating blog entries is a link for our new `/entry` route to the home page. 

<div class="code-description">
    <p>src/main/resources/templates/home.html</p>
</div>
<pre>
<code class="language-html">&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
&lt;head>
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0" />
    &lt;title>Java Blog&lt;/title>
&lt;/head>
&lt;body>
    &lt;h1>Blog Entries&lt;/h1>
    &lt;div th:each="entry : *{entries}">
        &lt;h2 th:text="${entry.title}">&lt;/h2>
        &lt;p th:text="${entry.content}">&lt;/p>
    &lt;/div>
    &lt;div>
        &lt;a href="/entry">New Post&lt;/a>
    &lt;/div>
&lt;/body>
&lt;/html>
</code>
</pre>
Since we already have a template to use for editing entries, we can create two new controller routes similar to the ones we just made for editing existing blog entries. 

<div class="code-description">
    <p>src/main/java/com/example/microblogspringboot/web/EntryController.java</p>
</div>
<pre>
<code class="language-java">package com.example.microblogspringboot.web;

// imports...
import org.springframework.web.bind.annotation.PathVariable;

@Controller
public class EntryController {
    
    // repository and other routes...

    @RequestMapping(value = "/entry/{id}", method = RequestMethod.GET)
    public String editEntry(@PathVariable(value = "id") Long entryId, Model model) {
        Entry entry = entryRepository.findOne(entryId);
        model.addAttribute("pageTitle", "Edit Entry");
        model.addAttribute("givenAction", "/entry/" + entryId);
        model.addAttribute("givenTitle", entry.getTitle());
        model.addAttribute("givenContent", entry.getContent());
        return "entry";
    }

    @RequestMapping(value = "/entry/{id}", method = RequestMethod.POST)
    public String updateEntry(@PathVariable(value = "id") Long entryId,
                              @RequestParam String title,
                              @RequestParam String content) {
        Entry entry = entryRepository.findOne(entryId);
        entry.setTitle(title);
        entry.setContent(content);
        entryRepository.save(entry);
        return "redirect:/";
    }
}
</code>
</pre>
For these routes, we're reading an entry ID from the URL path to get the entry we'll be editing from our database. The `findOne` method we're using is also inherited from Spring's `CrudRepository` class. Overall these routes are fairly similar to the ones we wrote for creating new entries.

The only change we'll need to make to our template is to add a link to the edit page for each entry:

<div class="code-description">
    <p>src/main/resources/templates/home.html</p>
</div>
<pre>
<code class="language-html">&lt;div th:each="entry : *{entries}">
    &lt;h2 th:text="${entry.title}">&lt;/h2>
    &lt;p th:text="${entry.content}">&lt;/p>
    &lt;a th:href="@{/entry/{id}(id = ${entry.id})}">edit&lt;/a>&lt;br />
&lt;/div>
</code>
</pre>
We're using the entity's ID to generate a unique link url for each blog entry. You can find a lot more information about how creating dynamic link urls in [Thymeleaf's documentation](http://www.thymeleaf.org/doc/articles/standardurlsyntax.html).

The only outstanding feature we'll need to implement to make this a full CRUD app is to allow users to delete blog entries. To make this tutorial as simple as possible, I've decided to use a `GET` request as opposed to `DELETE` for removing entries. By using `GET`, we can simply create a link in the UI and won't have to use any Javascript to send delete requests to the server. 

<div class="code-description">
    <p>src/main/java/com/example/microblogspringboot/web/EntryController.java</p>
</div>
<pre>
<code class="language-java">package com.example.microblogspringboot.web;

// imports...
import org.springframework.web.bind.annotation.PathVariable;

@Controller
public class EntryController {
    
    // repository and other routes...

    @RequestMapping(value = "/entry/delete/{id}", method = RequestMethod.GET)
    public String deleteEntry(@PathVariable(value = "id") Long entryId) {
        entryRepository.delete(entryId);
        return "redirect:/";
    }
}
</code>
</pre>

<div class="code-description">
    <p>src/main/resources/templates/home.html</p>
</div>
<pre>
<code class="language-html">&lt;div th:each="entry : *{entries}">
    &lt;h2 th:text="${entry.title}">&lt;/h2>
    &lt;p th:text="${entry.content}">&lt;/p>
    &lt;a th:href="@{/entry/{id}(id = ${entry.id})}">edit&lt;/a>&lt;br />
    &lt;a th:href="@{/entry/delete/{id}(id = ${entry.id})}">delete&lt;/a>
&lt;/div>
</code>
</pre>
We now have a fully working CRUD app, but it still looks pretty ugly. Let's add a CSS file to our `resources` directory to give our UI a bit of style. 

<div class="code-description">
    <p>src/main/resources/templates/static/css/main.css</p>
</div>
<pre>
<code class="language-css">body {
    font-family: "Arial";
    font-color: #2e2e2e;
}

h1 {
    text-align: center;
}

button {
    background-color: #8092ff;
    padding: 10px 20px;
    border-radius: 3px;
    border: none;
}

form {
    margin: auto;
    display: flex;
    flex-direction: column;
}

div {
    margin: auto;
    width: auto;
    max-width: 800px;
    margin-bottom: 30px;
}

input, textarea {
    font-size: 13px;
    padding: 5px;
    width: 100%;
}
</code>
</pre>
And that's it! We now have a very basic blog application. If your
app is not working for whatever reason, the source code for this tutorial is available
[here](https://github.com/scottmcallister/micro-blog-spring-boot).