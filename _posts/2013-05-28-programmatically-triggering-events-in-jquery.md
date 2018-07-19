---
title: Programmatically Triggering Events in jQuery
layout: post
permalink: /programmatically-triggering-events-in-jquery/
tags:
  - events
  - javascript
  - jquery
excerpt_separator: <!--more-->
old-comments: programmatically-triggering-events-in-jquery.html
comments: false
---

Recently, I made a tab-based menu system which loaded various content via Ajax whenever a user clicked on any of the tabs.

The only problem with this was, that when my page loaded the widget remained empty until the user had actually clicked something.

This left me wondering how best to simulate user interaction and initialize the widget programmatically.

<!--more-->

Luckily jQuery offers two methods which are suitable for the job: `.trigger()` and `.triggerHandler()`.

### .trigger()

This method executes all handlers and behaviours attached to the matched elements for the given event type.

Consider this example:

```html
<a class="clickMe" href="#">Link 1</a>
<a class="clickMe" href="#">Link 2</a>
<a class="clickMe" href="#">Link 3</a>

<script>
  $(".clickMe").on("click", function(){
    console.log("You clicked " + $(this).text());
  });

  $(".clickMe").trigger("click");
</script>
```

This would output:

```js
You clicked Link 1
You clicked Link 2
You clicked Link 3
```

`.trigger()` attempts to replicate the natural event as best as it can, but will not always replicate the browser's default actions.

For instance, in the above example the event handler bound to the anchor tags will be executed, but the browser will not be redirected, as would be the case with a normal click.

It's also worth noting that `.trigger()` returns the jQuery object, thus allowing chaining.

[http://api.jquery.com/trigger/](http://api.jquery.com/trigger/ "jQuery API docs: .trigger()")

## .triggerHandler()

This method is similar to .trigger(), with a few exceptions:

- It does not cause the default behavior of an event to occur (such as a form submission).
- It only affects the first matched element.
- Events created with `.triggerHandler()` do not bubble up the DOM hierarchy.
- It returns whatever value was returned by the last handler it caused to be executed. If no handlers are triggered, it returns `undefined`.

[http://api.jquery.com/triggerHandler/](http://api.jquery.com/triggerHandler/ "jQuery API docs: .triggerHandler()")

### So which one to use?

In this case I went with `.trigger()` to initialize my tab-based menu, as it is more succinct and I had no need of the additional behaviour offered by `.triggerHandler()`.

Here's an example of my tab-based navigation [without](http://hibbard.eu/demos/tabs/1/ "No initialization") `.trigger()` and [with](http://hibbard.eu/demos/tabs/2/ "Initialized using .trigger()") `.trigger()`.

As a final note, it is also possible to detect whether an event was triggered by the user, or whether it was triggered programmatically:

```js
$('#selecteor').on('click', function (e) {
  if (e.originalEvent === undefined) {
    console.log('triggered programmatically');
  } else {
    console.log('clicked by the user');
  }
})
```

[http://api.jquery.com/category/events/event-object/](http://api.jquery.com/category/events/event-object/ "jQuery API docs: Event Object")
