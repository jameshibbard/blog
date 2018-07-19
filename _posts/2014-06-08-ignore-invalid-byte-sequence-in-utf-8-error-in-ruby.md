---
title: 'Ignore "Invalid Byte Sequence in UTF-8" Error in Ruby'
layout: post
permalink: /ignore-invalid-byte-sequence-in-utf-8-error-in-ruby/
tags:
  - ruby
  - working with files
excerpt_separator: <!--more-->
comments: false
---

I wrote a simple Ruby script to parse text files and manipulate their content.

This is useful, for example, if you want to replace all occurrences of the phrase "Dr Jones" with "Prof. Jones" across a set of HTML files.

This was working great on Windows, but when I ran it under Linux, I started getting a "invalid byte sequence in UTF-8" error. This is how I solved it.

<!--more-->

## First the Script

This is the script I was using. It recursively walks through a directory tree, parsing any file which ends in .htm

```ruby
start_path = "/path/to/parent/directory/"

def change_text_file(path)
  file_contents = ""
  File.open(path,'r') do |file|
    while line = file.gets
      if line.match "whatever"
        line = line.sub("whatever", "new whatever")
      end
      file_contents += line
    end
  end

  File.open(path,'w') do |file|
    file.print inhalt
  end
end

def build_tree(start_path)
  Dir.open(start_path).entries.each do |file_name|
    path = File.join(start_path, file_name)
    next if /^\./ =~ file_name
    next if not /.htm/.match file_name and not File.directory?(path)
    File.directory?(path) ? build_tree(path) : change_text_file(path)
  end
end

build_tree(start_path)
```

And this is the error I was getting when I ran it:

```sh
simple_search_and_replace.rb:7:in 'match': invalid byte sequence in UTF-8 (ArgumentError)
  from simple_search_and_replace.rb:7:in `match'
  ...
```

The line in the file which was causing my script to choke contained a letter a with an umlaut (ä). Obviously it was an encoding issue.

I could have swapped the ä for the equivalent HTML entity (&amp;auml; in this case), but I didn't want to have to do that for every special characterin every single file.

Instead you can have Ruby "ignore" the invalid UTF-8 sequences by using `force_encoding` in conjunction with `String.encode` and specifying `#encoding: UTF-8` at the top of the file.

Note: [String#encode](http://www.ruby-doc.org/core-1.9.3/String.html#method-i-encode-21 "String#encode") was introduced in Ruby 1.9.3. If you have to work with an earlier version, you'll need to use the [Iconv](http://www.ruby-doc.org/stdlib-1.8.7/libdoc/iconv/rdoc/Iconv.html "Iconv library") library.

This left me with the following:

```ruby
#encoding: UTF-8
start_path = "/path/to/parent/directory/"

def change_text_file(path)
  file_contents = ""
  File.open(path,'r') do |file|
    while line = file.gets
      if line
           .force_encoding("ISO-8859-1")
           .encode("utf-8", replace: nil)
           .match "whatever"
        line = line.sub("whatever", "new whatever")
      end
      file_contents += line
    end
  end

  File.open(path,'w') do |file|
    file.print inhalt
  end
end

def build_tree(start_path)
  Dir.open(start_path).entries.each do |file_name|
    path = File.join(start_path, file_name)
    next if /^\./ =~ file_name
    next if not /.htm/.match file_name and not File.directory?(path)
    File.directory?(path) ? build_tree(path) : change_text_file(path)
  end
end

build_tree(start_path)
```

The double encoding trick is necessary, as I was reading ISO 8859-1 encoded files as UTF-8.

In this case, `force_encoding` forces an encoding conversion (and with it the check for invalid characters). Otherwise if the source string is already encoded in UTF-8, then just calling `encode('UTF-8')` is a no-op, and no checks are run.

### Reference

-  [Ruby Invalid Byte Sequence in UTF-8](https://stackoverflow.com/questions/9607554/ruby-invalid-byte-sequence-in-utf-8/ "Stack Overflow")
-  [Ruby 1.9: invalid byte sequence in UTF-8](http://stackoverflow.com/questions/2982677/ruby-1-9-invalid-byte-sequence-in-utf-8 "Stack Overflow")
-  [Fixing invalid UTF-8 in Ruby, revisited](http://po-ru.com/diary/fixing-invalid-utf-8-in-ruby-revisited/ "Fixing invalid UTF-8 in Ruby, revisited")

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.
