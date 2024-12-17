---
title: A Guide to the Pug HTML Template Preprocessor
layout: post
permalink: /a-beginners-guide-to-pug/
tags:
  - axios
  - express
  - node
  - pug
excerpt_separator: <!--more-->
canonical:
  url: 'https://www.sitepoint.com/a-beginners-guide-to-pug/'
---

<p class="callout originally-posted-elsewhere">This article was <a href="https://www.sitepoint.com/a-beginners-guide-to-pug/">first published on SitePoint</a> and has been republished here with permission.</p>

As web designers or developers, we likely all have to write our fair share of HTML. And while this is not the most difficult task, it can often feel a little boring or repetitive. HTML is also static, which means that if you want to display dynamic data (fetched from an API, for example), you invariably end up with a mishmash of HTML stings inside JavaScript. This can be a nightmare to debug and to maintain.


This is where [Pug](https://pugjs.org) comes in. <!--more--> Pug is a template engine for Node and for the browser. It compiles to HTML and has a simplified syntax, which can make you more productive and your code more readable. Pug makes it easy both to write reusable HTML, as well as to render data pulled from a database or API.

In this guide, I’ll demonstrate how to get up and running with Pug. We’ll start by installing it from npm, go over its basic syntax and then look at several examples of using JavaScript in Pug. Finally, we’ll explore a couple of Pug’s more advanced features by building a simple Node/Express project which uses Pug as its template engine.

## What’s a Template Engine and Why Do I Need One?

Before we start looking at Pug, let’s take a second to understand the concepts involved.

A template engine is a program which is responsible for compiling a template (that can be written using any one of a number of languages) into HTML. The template engine will normally receive data from an external source, which it will inject into the template it’s compiling. This is illustrated by the following diagram.

<div style="margin-bottom: 1.5em;">
<img style="margin-bottom: 0.5em;" src="https://uploads.sitepoint.com/wp-content/uploads/2023/09/1693957996web-documents.jpg" /><span style="display:block; font-size:0.8em; text-align:center; margin: 0;">Credit: Dreftymac, <a href="https://en.wikipedia.org/wiki/File:TempEngWeb016.svg">TempEngWeb016</a>, <a href="https://creativecommons.org/licenses/by-sa/3.0/legalcod">CC BY-SA 3.0</a></span>
</div>

This approach allows you to reuse static web page elements, while defining dynamic elements based on your data. It also facilitates a separation of concerns, keeping your application logic isolated from your display logic.

You’re more likely to benefit from a template engine if your site or web application is data driven — such as a staff directory for administering employees, a web store that lists various products for users to buy, or a site with dynamic search functionality.

You won’t need a template engine if you’re fetching a small amount of data from an API (in which case you can just use JavaScript’s native [template strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)), or if you’re making a small static site.

## A Little History

It’s also worth noting that Pug used to be called Jade until it was forced to change its name due to a trademark claim in 2015. The name change took effect with version 2.0.

There’s still a lot of Jade-related material available online. And while some of it’s probably still quite valid, the fact that the name change coincided with a major version bump means that Pug’s syntax has several differences, deprecations, and removals compared to its predecessor. These are documented [here](https://github.com/pugjs/pug/issues/2305).

If you’re interested in finding out more, you can read the original name change announcement in [this GitHub issue](https://github.com/pugjs/pug/issues/2184). Otherwise, just be sure to add the word “template” to your Pug-related Google searches to avoid the results being full of pooches.

## Installing Pug

Before we can get to writing some Pug, we’ll need to install Node, npm (which comes bundled with Node) and the [pug-cli package](https://www.npmjs.com/package/pug-cli).

There’s a couple options for installing Node/npm. Either head on over to the [project’s home page](https://nodejs.org/en/download/) and download the correct binaries for your system, or use a version manager such as [nvm](https://github.com/creationix/nvm). I would recommend using a version manager where possible, as this will allow you to install different Node versions and switch between them at will. It will also negate a bunch of potential permissions errors.

You can check out our tutorial “[Installing Multiple Versions of Node.js Using nvm](https://www.sitepoint.com/quick-tip-multiple-versions-node-nvm/)” for a more in-depth guide.

Once Node and npm are installed on your system, you can install the `pug-cli` package like so:

```bash
npm i -g pug-cli
```

You can check that the install process ran correctly by typing `pug --version` into a terminal. This will output the version of Pug and the version of the CLI that you have installed.

At the time of writing, this was as follows:

```bash
$ pug --version
pug version: 2.0.3
pug-cli version: 1.0.0-alpha6
```

### Syntax Highlighting in Your Editor

If your editor doesn’t offer syntax highlighting for Pug, it’d be a good idea to look for an extension to add this functionality.

I’m currently using Sublime Text 3 and, out of the box, this is what a `.pug` file looks like:

![Pug without syntax highlighting](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2019/03/1551450047pug-01.png)

To remedy this, one can install the [Sublime Pug package](https://packagecontrol.io/packages/Pug):

![Pug with syntax highlighting](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2019/03/1551450048pug-02.png)

Syntax highlighting will make it much easier to work with Pug files, especially those of any length.

### Try Pug without Installing

If you’d like to follow along with the simpler examples in this tutorial, you can also run them in various online code playgrounds.

[CodePen](https://codepen.io), for example, has Pug support baked right in. Simply create a new pen, then select _Settings_ > _HTML_ and choose Pug as your preprocessor. This will allow you to enter Pug code into the HTML pane and see the result appear in real time.

As an added bonus, you can click on the down arrow in the HTML pane and select _View Compiled HTML_ to see the markup that Pug has generated.

## Pug’s Basic Syntax

Now that we’ve got Pug installed, let’s try it out. Create a new directory named `pug-examples` and change into it. Then create a further directory called `html` and a file called `index.pug`:

```bash
mkdir -p pug-examples/html
cd pug-examples
touch index.pug
```

*Note: the `touch` command is Linux/macOS specific. Windows users would do `echo.> index.pug` to achieve the same thing.*

The way this is going to work is that we’ll write our Pug code in `index.pug` and have the `pug-cli` watch this file for changes. When it detects any, it will take the contents of `index.pug` and render it as HTML in the `html` directory.

To kick this off, open a terminal in the `pug-examples` directory and enter this:

```bash
pug -w . -o ./html -P
```

You should see something like the following:

```bash
watching index.pug
rendered /home/jim/Desktop/pug-examples/html/index.html
```

*Note: in the above command, the `-w` option stands for watch, the dot tells Pug to watch everything in the current directory, `-o ./html` tells Pug to output its HTML in the `html` directory and the `-P` option prettifies the output.*

Now let’s create the page from the screenshot above (the one complaining about the lack of syntax highlighting). Enter the following into `index.pug`:

```haml
doctype html
html(lang='en')
 head
   title Hello, World!
 body
   h1 Hello, World!
   div.remark
     p Pug rocks!
```

Save `pug.index` and then inspect the contents of `./html/index.html`. You should see the following:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Hello, World!</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
    <div class="remark">
      <p>Pug rocks!!</p>
    </div>
  </body>
</html>
```

Not bad, eh? The Pug CLI has taken our Pug file and rendered it as regular HTML.

This example serves to highlight a couple of important points about Pug. Firstly, it is **whitespace sensitive**, which means that Pug uses indentation to work out which tags are nested inside each other. For example:

```haml
div.remark
  p Pug rocks!!
```

The code above produces this:

```html
<div class="remark">
  <p>Pug rocks!!</p>
</div>
```

Now take this code:

```haml
div.remark
p Pug rocks!!
```

This produces the following:

```html
<div class="remark"></div>
<p>Pug rocks!!</p>
```

It doesn’t really matter what level of indentation you use (you can even use tabs if you have to), but it’s highly recommended that you keep the level of indentation consistent. In this article I’ll be using two spaces.

Secondly, **Pug doesn’t have any closing tags**. This will obviously save you a fair few keystrokes and affords Pug a clean and easy-to-read syntax.

Now that we’ve got a handle on some basic Pug, let’s quickly go over its syntax. If any of this seems confusing, or you’d like to go more in-depth, be sure to consult [the project’s excellent documentation](https://pugjs.org/api/getting-started.html).

### DOCTYPE

You can use Pug to generate a number of document type declarations.

For example `doctype html` will compile to `<!DOCTYPE html>`, the standard HTML5 doctype, whereas `doctype strict` will give us `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">`.  Pug will do its best to ensure that its output is valid for the document type.

### Tags

As mentioned, Pug doesn’t have any closing tags and relies on indentation for nesting. This might take a small amount of getting used to, but once you do, it makes for clean and readable code. By way of an example:

```haml
nav
  navbar-default  div
    h1 My Website!
  ul
    li
      a Home
    li
      a Page 1
    li
      a Page 2
  input
```

The code above compiles to this:

```html
<nav>
  <div>
    <h1>My Website!</h1>
  </div>
  <ul>
    <li><a>Home</a></li>
    <li><a>Page 1</a></li>
    <li><a>Page 2</a></li>
  </ul>
  <input/>
</nav>
```

Notice that Pug is smart enough to close any self-closing tags (such as the `<input />` element) for us.

### Classes, IDs and Attributes

Classes and IDs are expressed using a `.className` and  `#IDname` notation. For example:

```haml
nav#navbar-default
  div.container-fluid
    h1.navbar-header My Website!
```

Pug also offers us a handy shortcut. If no tag is specified, it will assume a `<div>` element:

```haml
nav#navbar-default
  .container-fluid
    h1.navbar-header My Website!
```

Both of these compile to:

```html
<nav id="navbar-default">
  <div class="container-fluid">
    <h1 class="navbar-header">My Website!</h1>
  </div>
</nav>
```

Attributes are added using brackets:

```haml
  ul
    li
      a(href='/') Home
    li
      a(href='/page-1') Page 1
    li
      a(href='/page-2') Page 2

  input.search(
    type='text'
    name='search'
    placeholder='Enter a search term...'
  )
```

This results in the following:

```html
<ul>
  <li><a href="/">Home</a></li>
  <li><a href="/page-1">Page 1</a></li>
  <li><a href="/page-2">Page 2</a></li>
</ul>
<input class="search" type="text" name="search" placeholder="Enter a search term..."/>
```

There’s a lot more to say about attributes. For example, you could use JavaScript to include variables in your attributes, or assign an array of values to an attribute. We’ll get on to using JavaScript in Pug in the next section.

### Plain Text and Text Blocks

Pug provides various methods for adding plain text directly into the rendered HTML.

We’ve already seen how to add plain text inline:

```haml
h1.navbar-header My Website! We can write anything we want here …
```

Another way is to prefix a line with a pipe character (`|`):

```haml
p
  | You are logged in as
  | user@example.com
```

This gives us the following:

```html
<p>
  You are logged in as
  user@example.com
</p>
```

When dealing with large blocks of text, you can just ad a dot `.` right after the tag name, or after the closing parenthesis, if the tag has attributes:

```haml
p.
  Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
  tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
  veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
  commodo consequat.
```

This results in:

```html
<p>
  Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
  tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
  veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
  commodo consequat.
</p>
```

### Comments

Finally, comments can be added like so:

```haml
// My wonderful navbar
nav#navbar-default
```

This comment will be added to the rendered HTML:

```html
<!-- My wonderful navbar-->
<nav id="navbar-default"></nav>
```

You start a comment like so:

```haml
//- My wonderful navbar
nav#navbar-default
```

When you do this, the comment will remain in the Pug file but won’t appear in the HTML.

Comments must appear on their own line. Here, the comment will be treated as plain text:

```haml
nav#navbar-default // My wonderful navbar
```

Multiline comments are possible, too:

```haml
//
  My wonderful navbar
  It is just so, awesome!
nav#navbar-default
```

### Basic Syntax Demo

Below you can find a demo of a Bootstrap-style layout which demonstrates the techniques we’ve discussed so far:

<p class="codepen" data-height="600" data-theme-id="6441" data-default-tab="result" data-user="SitePoint" data-slug-hash="wOgKPr" style="height: 626px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="Basic Pug Demo">
  <span>See the Pen <a href="https://codepen.io/SitePoint/pen/wOgKPr/">
  Basic Pug Demo</a> by SitePoint (<a href="https://codepen.io/SitePoint">@SitePoint</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## Using JavaScript in Pug

One of the great things about Pug is the ability to run JavaScript in your templates. This makes it easy to insert variables into our templates, iterate over arrays and objects, conditionally render HTML, and much more.

### Buffered vs Unbuffered Code

This is an important distinction to be aware of before using JavaScript in Pug.

**Unbuffered code** starts with a minus (`-`). It doesn’t directly add anything to the output, but its values may be used from within Pug:

```haml
- const name = "Jim"
//- Now I can refer to a 'name' variable in my Pug code
```

**Buffered code**, on the other hand, starts with an equals (`=`). It evaluates a JavaScript expression and outputs the result.

```haml
p= 'Two to the power of ten is: ' + 2**10
```

The code above compiles to this:

```html
<p>Two to the power of ten is: 1024</p>
```

For reasons of security, buffered code is HTML escaped.

```haml
p= '<script>alert("Hi")</script>'
```

The code above compiles to this:

```html
<p>&lt;script&gt;alert(&quot;Hi&quot;)&lt;/script&gt;</p>p>
```

### Interpolation

String interpolation is the process of replacing one or more placeholders in a template with a corresponding value. As we’ve just seen, buffered input offers one method of doing this. Another is using `#{}`. Here, Pug will evaluate any code between the curly brackets, escape it, and render it into the template.

```haml
- const name = "jim"
p Hi #{name}
```

The code above compiles to this:

```html
<p>Hi jim</p>
```

As the curly brackets can contain any valid JavaScript expression, this opens up a bunch of possibilities:

```haml
- const name = "jim"
- //- Upcase first letter
p Hi #{name.charAt(0).toUpperCase() + name.slice(1)}
```

This compiles to:

```html
<p>Hi Jim</p>
```

It’s also possible to render unescaped values into your templates using `!{}`. But this is not the best idea if the input comes from an untrusted source.

*Note: when you want to assign the value held in a variable to an element’s attribute, you can omit the `#{}`. For example: `img(alt=name)`.*

### Iteration

Pug’s `each` keyword makes it easy to iterate over arrays:

```haml
- const employees = ['Angela', 'Jim', 'Nilson', 'Simone']
ul
  each employee in employees
    li= employee
```

This results in the following:

```html
<ul>
  <li>Angela</li>
  <li>Jim</li>
  <li>Nilson</li>
  <li>Simone</li>
</ul>
```

You can also use it to iterate over the keys in an object:

```haml
-
  const employee = {
    'First Name': 'James',
    'Last Name': 'Hibbard'
  }
ul
  each value, key in employee
    li= `${key}: ${value}`
```

This results in:

```html
<ul>
  <li>First Name: James</li>
  <li>Last Name: Hibbard</li>
</ul>
```

Pug also lets you provide an else block that will be executed if the array or object is empty:

```haml
- const employees = []
ul
  each employee in employees
    li= employee
  else
    li The company doesn't have any employees. Maybe hire some?
```

Finally, note that you can use `for` as an alias for `each`.

### Conditionals

Conditionals offer a very handy way of rendering different HTML depending upon the result of a JavaScript expression:

```haml
-
  const employee = {
    firstName: 'James',
    lastName: 'Hibbard',
    extn: '12345'
  }

#employee
  p= `${employee.firstName} ${employee.lastName}`
  p Extension:
    if employee.extn
      =employee.extn
    else
      | n/a
```

In this example, we’re checking whether the `employee` object has an `extn` property, then either outputting the value of that property (if it exists), or the text “n/a”.

### JavaScript in Pug Demo

Below you can find a demo of some of the techniques we’ve discussed in this section. This showcases Pug’s benefits somewhat more than the previous demo, as all we need to do to add further employees is to add further objects to our `sitePointEmployees` array.

<p class="codepen" data-height="650" data-theme-id="6441" data-default-tab="result" data-user="SitePoint" data-slug-hash="qvRmKJ" style="height: 671px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="JavaScript in Pug Demo">
  <span>See the Pen <a href="https://codepen.io/SitePoint/pen/qvRmKJ/">
  JavaScript in Pug Demo</a> by SitePoint (<a href="https://codepen.io/SitePoint">@SitePoint</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## A Hands-on Example

Now that we have a reasonable idea of Pug’s syntax and how it works, let’s finish off by building a small [Express.js](https://expressjs.com/) app to demonstrate a couple of Pug’s more advanced features.

The code for this example is available on [GitHub](https://github.com/sitepoint-editors/beginners-guide-to-pug).

*Note: if you’ve not used Express before, no worries. It’s a web framework for Node.js which provides a robust set of features for building web apps. If you’d like to find out more, check out our [getting started with Express tutorial](https://www.sitepoint.com/create-new-express-js-apps-with-express-generator/).*

First off, let’s create a new project and install Express:

```bash
mkdir pug-express
cd pug-express
npm init -y
npm i express
```

Next create an `app.js` file in the `pug-express` folder:

```bash
touch app.js
```

Then add the following:

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello, World!');
});

app.listen(3000, () => {
  console.log('Listening on port 3000...');
});
```

Here we’re declaring a route (`/`), which will respond to a GET request with the text “Hello, World!” We can test this in our browsers, by starting the server with `node app.js` and then visiting [http://localhost:3000](http://localhost:3000).

If you see something like this, then things have gone to plan:

![Hello, World!](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2019/03/1551890665pug-03.png)


### Adding Some Data

This Express app won’t do anything too spectacular. We’ll be building a simple staff directory which fetches a list of employees from a database and displays them in a table. For that to happen, we’ll need a database and some data.

However … installing and configuring a database is a little heavy handed for this small example, so I’m going to use a package called [json-server](https://github.com/typicode/json-server). This will allow us to create a `db.json` file which it will turn into a REST API that we can perform CRUD operations against.

Let’s install it:

```bash
npm i -g json-server
```

Now create the aforementioned `db.json` file in the project’s root:

```bash
touch db.json
```

Finally, we need some JSON to populate it. We’ll use the [Random User Generator](https://randomuser.me/), which is a free, open-source API for generating random user data. Twenty-five people should do for our example, so head over to `https://randomuser.me/api/?results=25` and copy the results into `db.json`.

Finally, start the server in a second terminal window with:

```bash
json-server --watch db.json -p=3001
```

This will cause json-server to start up on port 3001 and watch our database file for changes.

### Setting up Pug as the Template Engine

Express has excellent support for using Pug, so very little configuration is necessary.

First, let’s add Pug to our project:

```bash
npm i pug
```

Then in `app.js` we need to tell Express to use Pug:

```js
app.set('view engine', 'pug');
```

Next, create a `views` directory, then in the `views` directory, add an `index.pug` file:

```bash
mkdir views
touch views/index.pug
```

Add some content to that file:

```haml
doctype html
html(lang='en')
 head
   title Hello, World!
 body
   h1 Hello, World!
```

Then alter `app.js` like so:

```js
const express = require('express');
const app = express();
app.set('view engine', 'pug');

app.get('/', (req, res) => {
  res.render('index');
});

app.listen(3000, () => {
  console.log('Listening on port 3000...');
});
```

Finally, restart the Node server, then refresh your browser and you should see this:

![Hello, World! from Pug](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2019/03/1551896019pug-04.png)

And that’s it. You’re good to go.

### Building the Staff Directory

The next task on the list is to hand some data to the Pug template to display. To do that, we’ll need a method of fetching the data from the json-server. Unfortunately, the fetch API isn’t implemented in Node, so let’s use [axios](https://github.com/axios/axios), the popular HTTP client instead:

```bash
npm i axios
```

Then alter `app.js` like so:

```js
const express = require('express');
const axios = require('axios');
const app = express();

app.set('view engine', 'pug');

app.get('/', async (req, res) => {
  const query = await axios.get('http://localhost:3001/results');
  res.render('index', { employees: query.data });
});

app.listen(3000, () => {
  console.log('Listening on port 3000...');
});
```

There’s a couple of things going on here. We’ve turned our route handler into an [async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), so that we can wait for the employee data to be returned from json-server before handing it off to the template.

Then we render the index as before, but this time we pass it an object literal containing all of our data.

*Note: you have to restart the Node server every time you make a change to `app.js`. If this starts to get annoying, check out [nodemon](https://www.npmjs.com/package/nodemon), which will do this for you.*

Now for the Pug. Change `index.pug` to look like the following:

```haml
doctype html
html(lang='en')
  head
    title Staff Directory
    link(rel='stylesheet' href='https://cdn.jsdelivr.net/npm/semantic-ui@2.3.3/dist/semantic.min.css')
    style.
      table.ui.celled img { display: inline-block; }
      footer { margin: 35px 0 15px 0; text-align: center }
  body
    main#main
    h1.ui.center.aligned.header Staff Directory
    .ui.container
      table.ui.celled.table.center.aligned
        thead
          tr
            th Avatar
            th First Name
            th Last Name
            th Email
            th Phone
            th City
        tbody
          each employee in employees
            tr
              td
                img.ui.mini.rounded.image(src=employee.picture.thumbnail)
              td #{employee.name.first}
              td #{employee.name.last}
              td #{employee.email}
              td #{employee.phone}
              td #{employee.location.city}
        tfoot
          tr
            th(colspan='6')
    footer
      p © #{new Date().getFullYear()} My Company
```

There’s hopefully nothing surprising going on here. We’re using [semantic-ui-css](https://www.npmjs.com/package/semantic-ui-css) for some styling, as well as a couple of styles of our own.

Then, in the table body we’re iterating over the array of `employees` that we are passing in from `app.js` and outputting their details to a table.

At the bottom of the page is a footer with our copyright claim and the current year.

If you refresh the page now, you should see this:

![The Staff Directory](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2019/03/1551900592pug-05.png)


### Template Inheritance

This is pretty nice already, but to round things off, I’m going to demonstrate how to structure our views to offer maximum flexibility as the project grows.

Let’s start off by creating a `layout.pug` file in the `views` directory:

```bash
touch views/layout.pug
```

Then add the following:

```haml
doctype html
html
  head
    title Staff Directory
    link(rel='stylesheet' href='https://cdn.jsdelivr.net/npm/semantic-ui@2.3.3/dist/semantic.min.css')
    style.
      table.ui.celled img { display: inline-block; }
      footer { margin: 35px 0 15px 0; text-align: center }

  body
    main#main
    h1.ui.center.aligned.header Staff Directory
    .ui.container

      block content

    block footer
      footer
        p © #{new Date().getFullYear()} My Company
```

What we’ve done here is create a layout file than can be extended by other Pug files within our project. When you have a large number of Pug files, this saves a considerable amount of code.

The way this works is that we’ve defined two blocks of content (`block content` and `block footer`) that a child template may replace. In the case of the `footer` block, we’ve also defined some fallback content that will be rendered if the child template doesn’t redefine this block.

Now we can tell our `index.pug` file to inherit from our layout:

```haml
extends layout.pug

block content
  table.ui.celled.table.center.aligned
    thead
      tr
        th Avatar
        th First Name
        th Last Name
        th Email
        th Phone
        th City
    tbody
      each employee in employees
        tr
          td
            img.ui.mini.rounded.image(src=employee.picture.thumbnail)
          td #{employee.name.first}
          td #{employee.name.last}
          td #{employee.email}
          td #{employee.phone}
          td #{employee.location.city}
    tfoot
      tr
        th(colspan='6')
```

The result is the same as we had before, but the code now has a better structure.

### Mixins

Mixins allow you to create reusable blocks of Pug. We can use this to extract our table row into its own file.

Create a folder called `mixins` in  the `views` folder and in that folder create a file named `_tableRow.pug`:

```bash
mkdir views/mixins
touch views/mixins/_tableRow.pug
```

Mixins are declared using the `mixin` keyword. They are compiled to functions and can take arguments. Add the following to `views/mixins/_tableRow.pug`:

```haml
mixin tableRow(employee)
  tr
    td
      img.ui.mini.rounded.image(src=employee.picture.thumbnail)
    td #{employee.name.first}
    td #{employee.name.last}
    td #{employee.email}
    td #{employee.phone}
    td #{employee.location.city}
```

Now alter `index.pug` like so:

```haml
extends layout.pug
include mixins/_tableRow

block content
  table.ui.celled.table.center.aligned
    thead
      tr
        th Avatar
        th First Name
        th Last Name
        th Email
        th Phone
        th City
    tbody
      each employee in employees
        +tableRow(employee)
    tfoot
      tr
        th(colspan='6')
```

As you can see, we’re importing the mixin at the top of the file.  We then call it by prefixing its name with a plus symbol and pass it our `employee` object to display.

This is overkill for our little app, but it demonstrates a very useful feature of Pug which allows us to write reusable code.

## Conclusion

Well done if you’ve made it this far! We’ve covered a lot of ground in this tutorial. We’ve looked at installing Pug, its basic syntax, its JavaScript support and constructs for iteration and conditional rendering. Finally, we built a fully functioning Express app which pulls data from a remote source and feeds it to a Pug template.

There’s still a lot more that Pug can do. I’d encourage you to check out its [excellent documentation](https://pugjs.org/api/getting-started.html) and to just start using it in your projects. You can also use it with several modern JS frameworks, such as [React](https://github.com/pugjs/babel-plugin-transform-react-pu) or [Vue](https://vue-loader.vuejs.org/guide/pre-processors.html#pug), and it has even been [ported to several other languages](https://github.com/pugjs/pug#ports-in-other-languages).

If you’re looking for a challenge, why not try extending the employee directory to add the missing CRUD functionality. And if you get stuck with the syntax, don’t forget that [help is always at hand](https://html-to-pug.com/).

Otherwise, if you have any remarks or questions, I'd love to hear them in the comments below.
