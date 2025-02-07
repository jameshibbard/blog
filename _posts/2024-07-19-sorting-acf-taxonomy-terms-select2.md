---
title: Sorting Selected Taxonomy Terms Alphabetically in ACF’s Select2 Field
layout: post
permalink: /sorting-acf-taxonomy-terms-select2/
excerpt_separator: <!--more-->
tags:
  - acf
  - custom post types
  - php
  - wordpress
  - wordpress hooks
  - wordpress taxonomy
image_shadow: true

---

When using [Advanced Custom Fields (ACF)](https://www.advancedcustomfields.com/) to manage taxonomy terms in a WordPress custom post type, the selected terms often appear in the order they were added rather than alphabetically.

This can be frustrating, especially when dealing with a long list of terms. Without a clear order, finding the right term at a glance can become a little tricky.

Luckily, we can hook into ACF’s filters to modify how the terms are loaded and ensure they are sorted alphabetically.

<!--more-->

> Click here if you want to [jump straight to the solution](#final-code-implementation)

## Keyword Order in the Post Editor

Before writing any code, let's take a second to understand the problem.

Imagine we have a **Research Papers** custom post type and a **Keyword** taxonomy. Each paper is assigned multiple keywords using an [ACF taxonomy field](https://www.advancedcustomfields.com/resources/taxonomy/). These keywords help categorize papers based on topics, making it easier to filter and search.

However, when selecting keywords in the WordPress post editor, the order in which they appear is not alphabetical—they show up in the order they were added.

![Screenshot of an ACF taxonomy field labeled 'Keywords' in the WordPress post editor. The field contains multiple selected terms displayed as gray pill-shaped tags, but they are not sorted alphabetically.](https://res.cloudinary.com/hibbard/image/upload/v1738878332/01-keyword-taxonomy-unsorted_e1btij.png)

What we really want are the selected keywords to appear alphabetically. This is a more structured approach which makes it easier to find and manage terms, especially as the number of associated keywords grows.

![Screenshot of an ACF taxonomy field labeled 'Keywords' in the WordPress post editor. The field contains multiple selected terms displayed as gray pill-shaped tags, now sorted alphabetically: Adaptive Neural Networks, Bayesian Inference for Data Analysis, Computational Linguistics in AI, Deep Reinforcement Learning, Evolutionary Optimization Algorithms, Generative Adversarial Networks in Healthcare, Hybrid Quantum Computing Methods, NLP for Biomedical Texts, Statistical Modeling of Complex Systems, Zero-Shot Image Recognition.](https://res.cloudinary.com/hibbard/image/upload/v1738878332/02-keyword-taxonomy-sorted_mpozfm.png)

So how can we acheive this?

## Hooking into ACF’s Load Filter

In order to sort the keywords, we first need to hook into [acf/load_value](https://www.advancedcustomfields.com/resources/acf-load_value/), a filter that allows us to modify the value of an ACF field before it is loaded in the editor.

Let's start by adding the following to our `functions.php` file:

```php
function sort_keywords_alphabetically( $value, $post_id, $field ) {
  error_log($value);

  return $value;
}

// Replace 'keywords' with the ACF field name assigned to your taxonomy.
add_filter(
  'acf/load_value/name=keywords',
  'sort_keywords_alphabetically',
  10, 3
);
```

Here we are inspect the data we are dealing with before making any modifications. The function logs `$value`, which represents the currently selected terms in the taxonomy field.

> If you're not sure how to use error_log in WordPress, you can check [this Stack Overflow answer](https://stackoverflow.com/a/55515556/1136887).

The [add_filter()](https://developer.wordpress.org/reference/functions/add_filter/) function attaches our custom function to ACF’s data loading process. The third argument (10) defines the filter’s priority—higher numbers run later, although 10 is the default. The fourth argument (3) tells WordPress to pass three parameters (`$value`, `$post_id`, `$field`) to our function, allowing us to inspect and modify the field’s value.

As we can see from the output, ACF provides an array of term IDs rather than term objects.

```text
Array
(
  [0] => 296
  [1] => 297
  [2] => 293
  [3] => 288
  [4] => 295
  [5] => 289
  [6] => 291
  [7] => 294
  8] => 290
  [9] => 292
)
```

Before we can sort the terms alphabetically by name, we need to retrieve their full details.

Change the function like so:

```php
function sort_keywords_alphabetically( $value, $post_id, $field ) {
  $terms = get_the_terms( $post_id, 'keyword' );
  error_log(print_r($terms, true));

  return $value;
}
```

Now the function uses WordPress's [get_the_terms()](https://developer.wordpress.org/reference/functions/get_the_terms/) to fetch the terms associated with the post. It then logs them using `error_log()`.

```text
Array
(
  [0] => WP_Term Object
    (
      [term_id] => 288
      [name] => Adaptive Neural Networks
      [slug] => adaptive-neural-networks
      [term_group] => 0
      [term_taxonomy_id] => 288
      [taxonomy] => keyword
      [description] =>
      [parent] => 0
      [count] => 1
      [filter] => raw
    )
    ...
)
```
## Sorting the Selected Keywords

With the data structure understood, we can now sort the keywords alphabetically before they are displayed in the post editor.

Change the function like so:

```php
function sort_keywords_alphabetically( $value, $post_id, $field ) {
  $terms = get_the_terms( $post_id, 'keyword' );

  usort( $terms, fn( $a, $b ) => strcmp( $a->name, $b->name ) );

  return array_map( fn( $term ) => $term->term_id, $terms );
}
```

We are now using [usort()](https://www.php.net/manual/en/function.usort.php) to arrange the terms alphabetically by their name property. Once sorted, we extract the term IDs using `array_map()` and return them. This ensures that the data is in the format that ACF is expecting.

The keywords should now be displayed in alphabetical order within the post editor.

But before we call it a day, we should check whether the field being processed is `keywords` and whether it has a value:

```php
if ( 'keywords' !== $field['name'] || empty( $value ) ) {
  return $value;
}
```
This prevents the function from running on other fields and avoids unnecessary processing when no terms are selected. If the field isn't `keywords` or the value is empty, we simply return it as is.

## Final Code Implementation

Here’s the final version of our function. This ensures that the selected taxonomy terms in ACF taxonomy fields are sorted into in alphabetical order before they appear in the post editor.


```php
function sort_keywords_alphabetically( $value, $post_id, $field ) {
  if ( 'keywords' !== $field['name'] || empty( $value ) ) {
    return $value;
  }

  $terms = get_the_terms( $post_id, 'keyword' );

  usort( $terms, fn( $a, $b ) => strcmp( $a->name, $b->name ) );

  return array_map( fn( $term ) => $term->term_id, $terms );
}

// Replace 'keywords' with the ACF field name assigned to your taxonomy.
add_filter(
  'acf/load_value/name=keywords',
  'sort_keywords_alphabetically',
  10, 3
);
```

By hooking into the `acf/load_value` filter, we retrieve the selected terms, sort them by name, and return the ordered list of term IDs.

And that's it. I hope this post helps someone (even if it's only me when I next revisit my code).

If you have any questions, please leave a comment below and I'll do my best to help.
