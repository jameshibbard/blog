---
title: Make a Filterable Table With Vue.js
layout: post
permalink: /filterable-table-vuejs/
excerpt_separator: <!--more-->
tags:
  - vue.js
twitter:
  title: "Make a Filterable Table With Vue.js"
  description: "Leverage Vue's reactivity to build a filterable table which only displays the rows that match whatever text a user has entered into a text input."
  image_url: https://res.cloudinary.com/hibbard/image/upload/f_auto,w_800/v1584882694/stock/lens.jpg
---

One of the things I love about Vue is its unobtrusive reactivity system. Models are plain JavaScript objects and when you modify them, Vue automatically updates a page's HTML to reflect the change. This makes state management easy and intuitive.

In this tutorial, I'll demonstrate how to leverage Vue's reactivity to build a filterable table. This will only display the rows that match whatever text a user has entered into a text input. I'll also show you how to highlight the matches.

This might be useful to help users quickly find what they are looking for in a long table. Once you have understood how it works, you can easily adapt it to lists or anything else you need to filter.

<!--more-->

For the impatient, there is a demo of what we'll end up with [at the end of the article](#the-finished-thing-on-codepen) and also [on CodePen](https://codepen.io/James_Hibbard/pen/wvaEaom).

## Basic Setup

This is the skeleton HTML we will be using.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Filterable table</title>
  </head>
  <body>
    <div id="app"></div>

    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.11/dist/vue.js"></script>
    <script>
      const app = new Vue({
        el: '#app',
        data: {},
        methods: {},
        computed: {},
      });
    </script>
  </body>
</html>
```

As you can see we are pulling in Vue from a CDN, then rendering an empty Vue app in the div element with the ID of `app`.

Now let's get a table displaying. We'll add the data for the rows as a `data` property on the Vue instance and we'll use a [v-for directive](https://vuejs.org/v2/api/#v-for) to display each row.

Here's the HTML:

```vue
<div id="app">
  <table>
    <thead>
      <tr>
        <th>Department</th>
        <th>Employees</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(row, index) in rows" :key="`employee-${index}`">
        <td>{% raw %}{{ row.department }}{% endraw %}</td>
        <td>{% raw %}{{ [...row.employees].sort().join(', ') }}{% endraw %}</td>
      </tr>
    </tbody>
  </table>
</div>
```

And here's the JavaScript:

```js
const app = new Vue({
  el: '#app',
  data: {
    rows: [
      { department: 'Accounting', employees: ['Bradley', 'Jones', 'Alvarado'] },
      { department: 'Human Resources', employees: ['Juarez', 'Banks', 'Smith'] },
      { department: 'Production', employees: ['Sweeney', 'Bartlett', 'Singh'] },
      { department: 'Research and Development', employees: ['Lambert', 'Williamson', 'Smith'] },
      { department: 'Sales and Marketing', employees: ['Prince', 'Townsend', 'Jones'] }
    ]
  },
  methods: {},
  computed: {},
});
```

There are a couple of things to notice here.

We have declared an `index` variable inside the `v-for` directive, which we are using in our [key attribute](https://vuejs.org/v2/api/#key). This is used by Vue's virtual DOM algorithm to improve performance.

As the employees aren't listed in alphabetical order in the data property, we are using [.sort()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) to sort them as they are rendered to the page. As `.sort()` mutates the array, we are using the spread syntax to create a copy of the array first. If we didn't do this, Vue would complain about an infinite loop in a render function.

Finally, once we have sorted the array and joined the entries, we are converting it to a string before rendering it to the page.

At this point you should have a basic table displaying.

## Now Let's Add Some Filtering

First off, let's add a text input for the user to type into. We'll use a [v-model directive](https://vuejs.org/v2/api/#v-model) to create a two-way binding with a data property.

```vue
<div id="app">
  <table>
    ...
  </table>

  <input type="text"
         placeholder="Filter by department or employee"
         v-model="filter" />
</div>
```

Next, add the `filter` data property to the Vue instance:

```js
data: {
  filter:'',
  rows: [ ... ],
},
```

Finally, in order to filter the table for specific employees, we'll make use of Vue's [computed properties](https://vuejs.org/v2/guide/computed.html#Computed-Properties). These help you avoid putting too much logic in your templates. They are also cached (as long as none of their reactive dependencies changes), meaning better performance.

```js
computed: {
  filteredRows() {
    return this.rows.filter(row => {
      const employees = row.employees.toString().toLowerCase();
      const department = row.department.toLowerCase();
      const searchTerm = this.filter.toLowerCase();

      return department.includes(searchTerm) ||
        employees.includes(searchTerm);
    });
  }
},
````

Here, we're using JavaScript's native [filter method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) to return a new array containing any elements that match the search term. We're lowercasing everything to ensure that the search is case insensitive.

We'll also need to update the HTML to make use of our computed property:

```vue
<tr v-for="(row, index) in filteredRows" :key="`employee-${index}`">
  ...
</tr>
```

If you run the code at this point, you should have a table which you can filter by department and by employee.

## How to Highlight the Matches

As a final touch, let's highlight the text in the table rows that matches what the user has entered into the text input.

We can do this using a method which we'll call `highlightMatches`.

```vue
<tr v-for="(row, index) in filteredRows" :key="`employee-${index}`">
  <td v-html="highlightMatches(row.department)"></td>
  <td v-html="highlightMatches([...row.employees].sort().join(', '))"></td>
</tr>
```

Our method will return whatever text it is passed, with `<strong>` tags wrapped around any matches. So that Vue interprets the `<strong>` tags as HTML and doesn't simply output them to the page, we're using the [v-html directive](https://vuejs.org/v2/api/#v-html) in our template.

This is what the method looks like:

```js
methods: {
  highlightMatches(text) {
    const matchExists = text.toLowerCase().includes(this.filter.toLowerCase());
    if (!matchExists) return text;

    const re = new RegExp(this.filter, 'ig');
    return text.replace(re, matchedText => `<strong>${matchedText}</strong>`);
  }
},
```

First we are looking to see if the text the method is passed contains whatever the user has typed in. If there is no match, we are simply returning the text as is. This will deal with cases when, for example, the match is present in the employees column, but there is nothing to highlight in the department column. Again we are using `.toLowerCase()` to make things case insensitive.

Assuming a match exists, we are creating a rexeg with whatever the user has typed in. We are then returning the text with the matched part wrapped in `<strong>` tags.

## The Finished Thing on CodePen

And that's all there is to it. Here is the final result running on CodePen with a little styling applied.

<p class="codepen" data-height="400" data-theme-id="dark" data-default-tab="result" data-user="James_Hibbard" data-slug-hash="wvaEaom" style="height: 400px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Filterable Table With Vue.js ">
  <span>See the Pen <a href="https://codepen.io/James_Hibbard/pen/wvaEaom">
  Filterable Table With Vue.js </a> by James Hibbard (<a href="https://codepen.io/James_Hibbard">@James_Hibbard</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## Why Might This Not Be a Good Idea?

Before we end, it's worth mentioning that this approach has a couple of downsides.

- As we are using JavaScript to render the table to the page, this will give us an SEO hit. Although search bots can supposedly parse and execute JavaScript, I wouldn't like to bet on what they see when they find our table. If SEO is important to you, you will need to look into [server-side rendering](https://vuejs.org/v2/guide/ssr.html) with something like Nuxt.
- We are offering people with JavaScript turned off a bad experience. In an ideal world, we would offer them some kind of fallback. If you think that nowadays everyone has JavaScript enabled, look [here](https://kryogenix.org/code/browser/everyonehasjs.html).

## Conclusion

In this post I have demonstrated how to leverage Vue's reactivity system to build a filterable table in only a few lines of code.

If you have any questions or comments, I'd be glad hear them below.
