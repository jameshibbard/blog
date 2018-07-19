---
title: How to Embed Video in your WordPress Site Using HTML5
layout: post
permalink: /how-to-embed-video-in-your-wordpress-site-using-html5/
excerpt_separator: <!--more-->
tags:
  - html5
  - video
  - wordpress
  - wordpress plugins
comments: false
---

I have a [Panasonic HDC-SD800](http://www.panasonic.co.uk/html/en_GB/Products/HDC-SD800/Overview/6828016/index.html "Product page for /HDC-SD800") high definition video camera and I wanted to embed a couple of short film clips on a private website of mine, which runs on the latest version of WordPress (3.4.1 at time of writing).

I also wanted to embed these videos using HTML5 for those browsers which support it, and have a Flash fall-back for those which don't.

<!--more-->

The files come off of the camera in .m2ts format, so the first thing to do is to convert them into a container format which can be played by one of the modern browsers. Unfortunately, there is no one standard format (that would be too simple, eh?). The modern browsers support a combination of:

- H.264 video and AAC audio in an MP4 container
- Theora video and Vorbis audio in an Ogg container
- WebM

This is just a fleeting overview and the landscape is constantly changing. For a (much) more detailed overview see here: [http://fortuito.us/diveintohtml5/video.html#what-works](http://fortuito.us/diveintohtml5/video.html#what-works "Dive into HTML 5 - What works on the web")

## Encoding H.264 Video with HandBrake

[HandBrake](http://handbrake.fr/ "Handbrake homepage") is an open source application for encoding H.264 video for use on the web.

To convert my .m2ts file I did the following:

  * Select the _iPhone & iPod touch_ preset.
  * Make sure that the _Web optimized_ box is checked.
  * Select the maximum width and height of the encoded video ensuring that the _Keep Aspect Ratio_ is checked (I selected a width of 750)
  * In the video tab set the average bit rate to 1500 and check _2-Pass encoding_

Again, you can find a much more detailed description here: [http://fortuito.us/diveintohtml5/video.html#handbrake-gui](http://fortuito.us/diveintohtml5/video.html#handbrake-gui "Encoding H.264 Video with HandBrake")

## Encoding Ogg Video with Firefogg

I did this using the [Firefogg](http://firefogg.org/ "Firefogg homepage") extension for Firefox &#8211; a great extension which does all of the encoding client side. Here are the steps I took:

  * Select Ogg (Theora/Vorbis) format
  * Click _Advanced Options_
  * Enter the width and height of the encoded video (I selected 750 x 416)
  * Enter bit rate of 1500 kb/s
  * Enter quality 8
  * Select _2-Pass encoding_
  * Select an audio quality of 5.0

More indepth instructions can be found here: [http://fortuito.us/diveintohtml5/video.html#firefogg](http://fortuito.us/diveintohtml5/video.html#firefogg "Encoding Ogg Video with Firefogg")

## Encoding WebM with Firefogg

Excellent as it is, Firefogg can also encode WebM video, so just repeat the steps above, just select the format as being WebM (VP8/Vorbis) and a quality of 10 (the WebM format being considerably more compact than Ogg).

## A Suitable Plugin

Following these steps gives us all of the files we need to embed the video in the website. Now it's time to look for a suitable WordPress plugin. After much searching I settled for [VideoJS](http://wordpress.org/extend/plugins/videojs-html5-video-player-for-wordpress/ "VideoJS - HTML5 Video Player for WordPress"). You can install this in the usual way, then upload the files to the media library.

It was at this point that WorPress started complaining:

> "your_file.webm" has failed to upload due to an error
> Sorry, this file type is not permitted for security reasons.


To get around this, add this to your `wp-config.php` file:

```php
define('ALLOW_UNFILTERED_UPLOADS',true);
```

That will allow administrator level users to upload files without the file type restrictions.

So, with the files uploaded and the plugin activated, it is time to embed them in a post.

VideoJS does this using a tag, which accepts several parameters. These are explained on the plugin's installation page: [http://wordpress.org/extend/plugins/videojs-html5-video-player-for-wordpress/installation/](http://wordpress.org/extend/plugins/videojs-html5-video-player-for-wordpress/installation/ "Explanation of VideoJS parameters")

Suffice to say, I ended up with something like this:

```html
[video mp4="absolute path to m4v"
       ogg="absolute path to ogv"
       webm="absolute path to webm"
       preload="auto" autoplay="false" width="750" height="416"]
```

There is one final step before the video will play correctly, and that is to register the correct MIME types with the server. This helps the browsers deal correctly with the files they are served and render, for example, an mp4 file as a video and not a text file.

To do this add the following to the `.htaccess` file in the root of your WordPress install:

```apache
#Register MIME types
AddType video/x-m4v .m4v
AddType video/ogg .ogv
AddType video/mp4 .mp4
AddType video/webm .webm
```

And that's it. Using the VideoJS plugin and HTML 5 `<video>` element, your video will now play across all of the modern browsers. Also, the great thing about VideoJS is that it will serve up a Flash alternative to those browsers that don't support HTML 5 (e.g. IE 7 / IE 8).

The only other thing that might be worth doing is to make the video responsive. I added this functionality to my site when I saw that it previewed just fine on all desktop browsers, but that the video burst out of its container on the iPad.

To do this you should wrap the video tag in a <div> and give it a class of videoWrapper.

```html
<div class="videoWrapper">
</div>
```

and then copy the following css into your style.css file.:

```css
.videoWrapper {
  position: relative;
  padding-bottom: 55%; /* video dimensions - height/width */
  padding-top: 0px;
  height: 0;
  z-index: 1000;
}

video {
  position: absolute !important;
  top: 0;
  left: 0;
  width: 100% !important;
  height: 100% !important;
  z-index: 1;
}

video.video-js {
  z-index: 1000;
}

.video-js .vjs-controls {
  z-index: 1002;
}

.video-js .vjs-big-play-button {
  z-index: 1002;
}

.videoWrapper .video-js {
  position: absolute;
  top: 0;
  left: 0;
  width: 100% !important;
  height: 100% !important;
  z-index: 1;
  background: #000000;
}

.videoWrapper object,.videoWrapper embed {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100% !important;
  z-index: 1;
}

.vjs-spinner {
  display: none !important;
}

.video-js img.vjs-poster {
  height: 100% !important;
  width: 100% !important;
  z-index: 1;
  max-width: 100%;
}
```

One gotcha here, is the value for the bottom padding of the _videoWrapper_ div. This must be calculated on an individual basis and is the result in percent of dividing the height of your video by the length.

The credit for this code goes to Sarah Hills at Hexagon Webworks. She has written a blog post on the subject of making the VideoJS plugin responsive and it is well worth reading: [http://www.hexagonwebworks.com/2012/responsive-videos-updated/](http://www.hexagonwebworks.com/2012/responsive-videos-updated/ "Hexagon webworks: Responsive video")
