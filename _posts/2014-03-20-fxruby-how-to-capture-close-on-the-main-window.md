---
title: FXRuby – How to Capture Close on the Main Window
layout: post
permalink: /fxruby-how-to-capture-close-on-the-main-window/
tags:
  - fxruby
  - gui programming
  - ruby
  - ruby gems
excerpt_separator: <!--more-->
comments: false
---

I wrote a small FXRuby program which allows the user to do some simple file manipulation, then upload the altered files via FTP to a web server.

I ran into a small problem in that when the user clicked the _Close_ button (the X in the upper right-hand corner) of the main window, the application closed straight away.

Normally, this would be the desired behaviour, but in this case I wanted to intercept this message and do some application cleanup first (delete temporary files, make sure the user had saved their work etc).

Here's how I did it.

<!--more-->

```ruby
require 'fox16'
include Fox

class MyApp < FXMainWindow
  def initialize(app)
    @app = app
    super(app, "Test", :height => 150, :width => 350, :opts=> DECOR_ALL)
    self.connect(SEL_CLOSE, method(:on_close))
  end

  def create
    super
    show(PLACEMENT_SCREEN)
  end

  def on_close(sender, sel, event)
    q = FXMessageBox.question(@app, MBOX_YES_NO, "Sure?", "You sure?")
    if q == MBOX_CLICKED_YES
      return 0
    end
  end
end

FXApp.new do |app|
  MyApp.new(app)
  app.create
  app.run
end
```

The important thing is to associate the `SEL_CLOSE` message from the main window (i.e. self) with a separate handler method `on_close`.

Within the `on_close` method, any cleanup can be performed, before the application terminates.

In my example, I am returning zero from the `SEL_CLOSE` handler, which will cause the application to go ahead and exit if the user clicks "Yes"

## Taking It Further

As I am doing things within my app that have the potential to fail independently of me (such as logging into an FTP server), it makes life considerably easier to catch any errors in one place, rather than to have to try and deal with every error as it arises.

This lead me to come up with the following method of doing this from within the start-up block.

```ruby
FXApp.new do |app|
  begin
    editor = Editor.new(app)
    app.create
    app.run
  rescue => error
    if DEBUG
      dump(error)
    else
      FXMessageBox.warning(app, MBOX_OK, "Error!", "#{error}")
    end
  ensure
    editor.tidy_up
  end
end
```

This approach allows me to raise an exception from anywhere within the app, then output a detailed error message to the terminal if the app was started with the `debug` flag, or alternatively, display a simplified error message to the user in an FXMessageBox.

An example of use:

```ruby
@con.establish(@remote_dir, true)
raise ("Application in use") if @con.lock_present?
```

Also, by using an `ensure` block, the `tidy_up` method is guaranteed to get called and cleanup will be carried out.

## A Neat Trick

In the course of working out the above, I learned that method definitions are implicitly also exception blocks, so instead of writing:

```ruby
def foo
  begin
    # ...
  rescue
    # ...
  end
end
```

you can just do:

```ruby
def foo
  # ...
rescue
  # ...
end
```

or:

```ruby
def foo
  # ...
ensure
  # ...
end
```

See: [Begin, Rescue and Ensure in Ruby?](http://stackoverflow.com/questions/2191632/begin-rescue-and-ensure-in-ruby "The general flow of begin/rescue/else/ensure/end")

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.
