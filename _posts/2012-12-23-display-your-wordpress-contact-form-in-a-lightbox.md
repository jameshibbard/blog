---
title: Display Your WordPress Contact Form in a Lightbox
layout: post
permalink: /display-your-wordpress-contact-form-in-a-lightbox/
tags:
  - forms
  - wordpress
  - wordpress plugins
old-comments: display-your-wordpress-contact-form-in-a-lightbox.html
comments: false
---

Recently, a client asked me to have their contact form display in a Lightbox (as opposed to on a separate page). I happily implemented this for them and the end result looked pretty snazzy, so I thought I'd write up how I did it.

The first thing you're going to need is a contact form and this is best achieved by using a plugin. So, from your WordPress dashboard select _Plugins_ > _Add New_ and enter _Contact Form 7_ into the search box. All things being equal, you should see a plugin of the same name at the top of your search results. Install it.

I've tried various contact form plugins in WordPress and this is by far my favourite. It is lightweight, super easy to use and totally configurable to boot. If you fancy reading more about it, the project's homepage can be found here: [http://contactform7.com/](http://contactform7.com/ "Contact Form 7 - Just another contact form plugin for WordPress. Simple but flexible.")

While you're installing plugins, there's one more we're going to need. This one is called **Form Lightbox** and can be obtained in the same way as described above. This plugin will enable the Lightbox effect for our contact form. Its homepage can be found here: [http://www.myphpmaster.com/form-lightbox/](http://www.myphpmaster.com/form-lightbox/ "Tutorial: WordPress Plugin – Form Lightbox")

Now, if you look at your Dashboard menu in your WordPress backend, you should notice that Contact Form 7 has created a new menu item named _Contact_. If you click on this, you will see an overview of your various contact forms. Currently we only have one (named _Contact form 1_ or similar).

The form will have a shortcode associated with it similar to:

```html
[contact-form-7 id="401" title="Contact form 1"]
```

If you were to paste this shortcode into any page on your WordPress site, this would generate a fully functional contact form on that page. As it is, we'll just copy the code for now, as we'll need it later.

Now we need to decide where we'll put our new contact form. It would, of course, be possible to have a link pointing to the contact form, which you could then place in your sidebar or in your footer.

Here is the shortcode you need to do that. It is essentially the Contact form 7 shortcode, wrapped in Form Lightbox's own shortcode:

```html
[formlightbox title="Send me a message" text="Contact me"]
  [contact-form-7 id="401" title="Contact form 1"]
[/formlightbox]
```

You can find a list of Form Lightbox's other parameters, as well as some further examples of use, here: [http://www.myphpmaster.com/form-lightbox-demo/](http://www.myphpmaster.com/form-lightbox-demo/ "Form Lightbox Demo"). Form Lightbox also has a slew of configuration options, which can be found via _Settings_ > _Form Lightbox_ in your WordPress backend.


## Adding Form Lightbox to Your Main Navigation

However, my client didn't want a link, he wanted the Lightbox functionality attached to the contact button in his main navigation.

To implement this I had to edit the file _header.php_, in which I had hard-coded the menu. The mark-up for the menu looked something like this:

```html
<ul id="myMenu">
  <li id="blog"><a href="/blog">Blog</a></li>
  <li id="humor"><a href="/humor">Humor</a></li>
  <li id="consulting"><a href="/consulting">Consulting</a></li>
  <li id="contact"><a href="/contact">Contact</a></li>
</ul>
```

As the kind of shortcodes listed above are specific to WordPress' code editor, I had to use the method `do_shortcode()` to make them render in a template. The code then looked like this:

```php
<ul id="myMenu">
  <li id="blog"><a href="/blog">Blog</a></li>
  <li id="humor"><a href="/humor">Humor</a></li>
  <li id="consulting"><a href="/consulting">Consulting</a></li>
  <li id="contact"><a href="/contact">
    <?php echo do_shortcode('
        [formlightbox title="Send me a message" text="Contact me"]
          [contact-form-7 id="401" title="Contact form 1"]
        [/formlightbox]
      ');
    ?>
  </li>
</ul>
```

Job done!

### Reference:

- Form Lightbox on WordPress plugin directory: [http://wordpress.org/extend/plugins/form-lightbox/](http://wordpress.org/extend/plugins/form-lightbox/ "Form Lightbox")
- Contact form 7 on WordPress plugin directory: [http://wordpress.org/extend/plugins/contact-form-7/](http://wordpress.org/extend/plugins/contact-form-7/ "Contact Form 7")
- A fantastic tutorial on how to achieve the same effect when not using WordPress: [http://designwoop.com/2012/07/tutorial-coding-a-jquery-popup-modal-contact-form/](http://designwoop.com/2012/07/tutorial-coding-a-jquery-popup-modal-contact-form/ "Tutorial: Coding a jQuery Popup Modal Contact Form")
