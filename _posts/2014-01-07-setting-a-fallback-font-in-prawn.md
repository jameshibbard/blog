---
title: Setting a Fallback Font in Prawn
layout: post
permalink: /setting-a-fallback-font-in-prawn/
tags:
  - pdf generation
  - prawn
  - rails
  - ruby
  - ruby gems
  - working with files
excerpt_separator: <!--more-->
comments: false
---

At work, one of the applicants to our programme submitted parts of her application in Russian (despite it being an English speaking programme).

The database could handle this fine, as it stores entries using the UTF-8 character set, but the PDF generation part of our application refused to cooperate (just displaying a bunch of underscores).

We use the [Prawn library](http://prawn.majesticseacreature.com/ "Prawn - Fast, Nimble PDF Writer for Ruby") to generate our PDFs, so I set about finding a way to make Prawn play nice with the Cyrillic characters.

<!--more-->

## Introducing Fallback Fonts

Luckily Prawn enables the declaration of fallback fonts for those glyphs that may not be present in the desired font.

In our case we were using Prawn's default font (Helvetica) which definitely doesn't support Cyrillic characters, so I had to find a font that did.

After a bit of Googling, I came across a site called Font Squirrel which offered [a list of fonts that support the Cyrillic language](http://www.fontsquirrel.com/fonts/list/language/cyrillic "Font Squirrel - Free Font Utopia").

I opted for [DejaVuSans](http://www.fontsquirrel.com/fonts/list/foundry/dejavu-fonts "Fonts by DejaVu fonts") and downloaded four .ttf files (normal, italic, bold, bold-italic) which I placed in the same directory as my PDF generation script.

I then had to inform Prawn of the presence of these fonts:

```ruby
font_families.update(
  "DejaVuSans" => {
    :normal => "#{File.dirname(__FILE__)}/DejaVuSans.ttf",
    :bold => "#{File.dirname(__FILE__)}/DejaVuSans-Bold.ttf",
    :italic => "#{File.dirname(__FILE__)}/DejaVuSans-Oblique.ttf",
    :bold_italic => "#{File.dirname(__FILE__)}/DejaVuSans-BoldOblique",
  }
)
```

With that done, I had to decide whether to use them on a case by case basis (with any of the `text` or `text box` methods), or as a document-wide fallback.

Document-wide seemed to be the way to go:

```ruby
fallback_fonts(["DejaVuSans"])
```

After that I uploaded everything, restarted the Rails app and that was it – we can now accept applications in Russian!

## A Small Standalone Example

To further illustrate the point, here's a small Ruby script that generates a PDF containing Cyrillic characters :

```ruby
#!/usr/bin/env ruby
#encoding: UTF-8
require "prawn"

Prawn::Document.generate("hello_world.pdf") do
  font_families.update(
    "DejaVuSans" => {
      :normal => "#{File.dirname(__FILE__)}/DejaVuSans.ttf",
      :bold => "#{File.dirname(__FILE__)}/DejaVuSans-Bold.ttf",
      :italic => "#{File.dirname(__FILE__)}/DejaVuSans-Oblique.ttf",
      :bold_italic => "#{File.dirname(__FILE__)}/DejaVuSans-BoldOblique",
    }
  )
  fallback_fonts(["DejaVuSans"])

  text "Bob says, 'Hello World'"
  text "Igor says, 'привет мир'"
end
```

### Reference

[Prawn Manual (see 'text/fallback_fonts.rb')](http://prawn.majesticseacreature.com/manual.pdf "Prawn by example - a self generating PDF manual")
