---
title: How to Loop Through a JSON Response in JavaScript
layout: post
permalink: /loop-through-json-response-javascript/
tags:
  - json
  - javascript
excerpt_separator: <!--more-->
canonical:
  url: 'https://www.sitepoint.com/loop-through-json-response-javascript/'
---

<p class="callout originally-posted-elsewhere">This article was <a href="https://www.sitepoint.com/loop-through-json-response-javascript/">first published on SitePoint</a> and has been republished here with permission.</p>

When fetching data from a remote server, the server's response will often be in JSON format. In this quick tip, I'll demonstrate how you can use JavaScript to parse the server's response, so as to access the data you require.

<!--more-->

This process will typically consist of two steps: decoding the data to a native structure (such as an array or an object), then using one of JavaScript's in-built methods to loop through that data structure. In this article, I'll cover both steps, using plenty of runnable examples.

## What is JSON?

Before we look at how to deal with JSON, let's take a second to understand what it is (and what it isn't).

JSON stands for <strong>J</strong>ava<strong>S</strong>cript <strong>O</strong>bject <strong>N</strong>otation. It's a language-independent, text-based format, which is commonly used for transmitting data in web applications. JSON was inspired by the JavaScript Object Literal notation, but there are differences between the two. For example, in JSON keys must be quoted using double quotes, while in object literals this is not the case.

There are two ways data can be stored in JSON:

- a collection of name/value pairs (aka a JSON object)
- an ordered list of values (aka a JSON array)

When receiving data from a web server, the data is always a string, which means that it's your job to convert it into a data structure you can work with.

If you'd like to find out more about how JSON works, please visit the [JSON website](https://www.json.org/json-en.html).

## Fetching JSON from a Remote API

In the following examples, we'll use the fantastic [icanhazdadjoke API](https://icanhazdadjoke.com/api). As you can read in its documentation, making a GET request where the `Accept` header is set to `application/json` will see the API return a JSON payload.

Let's start with a simple example:

```js
const xhr = new XMLHttpRequest();
xhr.onreadystatechange = () => {
  if (xhr.readyState === XMLHttpRequest.DONE) {
    console.log(typeof xhr.responseText);
    console.log(xhr.responseText);
  }
};
xhr.open('GET', 'https://icanhazdadjoke.com/', true);
xhr.setRequestHeader('Accept', 'application/json');
xhr.send(null);

// string
// {"id":"daaUfibh","joke":"Why was the big cat disqualified from the race? Because it was a cheetah.","status":200}
```

As we can see, the server returned us a string. We'll need to parse this into a JavaScript object before we can loop through its properties. We can do this with [JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse):

```js
if (xhr.readyState === XMLHttpRequest.DONE) {
  const res = JSON.parse(xhr.responseText);
  console.log(res);
};

// Object { id: "fiyPR7wPZDd", joke: "When does a joke become a dad joke? When it becomes apparent.", status: 200 }
```

Once we have our response as a JavaScript object, there are a number of methods we can use to loop through it.

### Use a `for...in` Loop

A [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) loop iterates over all enumerable properties of an object:

```js
const res = JSON.parse(xhr.responseText);

for (const key in res){
  if(obj.hasOwnProperty(key)){
    console.log(`${key} : ${res[key]}`)
  }
}

// id : H6Elb2LBdxc
// joke : What's blue and not very heavy?  Light blue.
// status : 200
```

Please be aware that `for...of` loops will iterate over the entire prototype chain, so here we're using `hasOwnProperty` to ensure that the property belongs to our `res` object.

### Use `Object.entries`, `Object.values` or `Object.entries`

An alternative approach to above is to use one of [Object.keys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys), [Object.values()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values) or [Object.entries()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries). These will return an array which we can then iterate over.

Let's take a look at using `Object.entries`. This returns an array of the key/value pairs of the object we pass it:

```js
const res = JSON.parse(xhr.responseText);

Object.entries(res).forEach((entry) => {
  const [key, value] = entry;
  console.log(`${key}: ${value}`);
});

// id: SvzIBAQS0Dd
// joke: What did the pirate say on his 80th birthday? Aye Matey!
// status: 200
```

Note that the `const [key, value] = entry;` syntax is an example of [array destructuring](https://www.sitepoint.com/es6-destructuring-assignment/) that was introduced to the language in ES2015.

This is much more concise, avoids the aforementioned prototype problem, and is my preferred method of looping through a JSON response.

## Using the Fetch API

While the method above using the `XMLHttpRequest` object works just fine, it can get unwieldy pretty quickly. We can do better.

The [Fetch API](https://www.sitepoint.com/introduction-to-the-fetch-api/) is a Promise-based API, which enables a cleaner, more concise syntax and helps keep you out of callback hell. It provides a `fetch()` method defined on the `window` object, which you can use to perform requests. This method returns a Promise that you can use to retrieve the response of the request.

Let's rewrite our previous example to use it:

```javascript
(async () => {
  const res = await fetch('https://icanhazdadjoke.com/', {
    headers: { Accept: 'application/json' },
  });
  const json = await res.json();
  Object.entries(json).forEach(([key, value]) => {
    console.log(`${key}: ${value}`);
  });
})();

// id: 2wkykjyIYDd
// joke: What did the traffic light say to the car as it passed? "Don't look I'm changing!"
// status: 200
```

The Fetch API returns a [response stream](https://developer.mozilla.org/en-US/docs/Web/API/Response). This is not JSON, so instead of trying to call `JSON.parse()` on it, we'll need to use its [response.json()](https://developer.mozilla.org/en-US/docs/Web/API/Body/json) function. This returns a Promise that resolves with the result of parsing the response's body text as JSON.

## Dealing with an Array

As mentioned at the top of the article, an ordered list of values (aka an array), is valid JSON, so before we finish, let's examine how to deal with such a response.

For the final example, we'll use [GitHub's REST API](https://docs.github.com/en/rest) to get a list of a user's repositories:

```js
(async () => {
  async function getRepos(username) {
    const url = `https://api.github.com/users/${username}/repos`;

    const response = await fetch(url);
    const repositories = await response.json();

    return repositories;
  }

  const repos = await getRepos('jameshibbard');
  console.log(repos);
})();

// Array(30) [ {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, … ]
```

As you can see, the API has returned an array of objects. To access each of the individual objects, we can use a regular `forEach` method:

```js
repos.forEach((repo) => {
  console.log(`{$repo.name} has ${repo.stargazers_count} stars`);
});

// Advanced-React has 0 stars
// angular2-education has 0 stars
// aurelia-reddit-client has 3 stars
// authentication-with-devise-and-cancancan has 20 stars
// ...
```

Alternatively, you can of course use any of the methods discussed above to loop through all of the object's properties and log them to the console:

```js
repos.forEach((repo) => {
  Object.entries(repo).forEach(([key, value]) => {
    console.log(`${key}: ${value}`);
  });
});

// name: Advanced-React
// full_name: jameshibbard/Advanced-React
// private: false
// ...
```

## Conclusion

In this quick tip, we've looked at what JSON is. I've demonstrated how to parse a JSON response from a server into a native data structure (such as an array or an object), and how to loop through such a structure, so as to access the data it contains.

If you're having trouble with anything presented in this article, why not stop by [SitePoint's Forums](https://www.sitepoint.com/community/c/javascript/33), where there are plenty of friendly people to help you out.
