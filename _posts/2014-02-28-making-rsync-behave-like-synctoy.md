---
title: Making rsync Behave like SyncToy
layout: post
permalink: /making-rsync-behave-like-synctoy/
tags:
  - linux
  - rsync
  - working with files
excerpt_separator: <!--more-->
old-comments: making-rsync-behave-like-synctoy.html
comments: false
---

Microsoft have produced a very handy backup tool for Windows in the form of [SyncToy](http://www.microsoft.com/en-us/download/details.aspx?id=15155 "SyncToy 2.1").

It is intuitive and easy to use, so when I made the switch from Windows to Linux as my main operating system, this was something I sorely missed.

I quickly stumbled upon [rsync](http://rsync.samba.org/ "The rsync web pages") as the de-facto backup program for Linux, but must admit I was somewhat overwhelmed with the plethora of configuration options it ships with, as well as the tremendously verbose output it produces.

Here's how to tame it somewhat.

<!--more-->

## Why Is SyncToy so Good?

My main use case for SyncToy was to make [incremental backups](http://en.wikipedia.org/wiki/Incremental_backup "If you don't know what this is, you should find out") of my data. Let me explain:

I have my OS installed on one partition of my hard drive and keep all of my data on a second partition. I keep a mirrored copy of my data partition on an external hard drive and used SyncToy to keep my data partition and my hard drive backup in sync.

In SyncToy speak, this operation is called 'echo' and is defined thus:

-  _Echo_ looks for changes (file modifications, new files, renames, deletes) in the left folder and makes them in the right folder (one-way sync).

If you're interested in what other operations SyncToy can perform, then check out its [Wikipedia page](http://en.wikipedia.org/wiki/SyncToy "SyncToy on Wikipedia").

To illustrate this further, my left hand folder was `D:\files` and my right hand folder was `F:\backup\files`.

Once I had set up the folder pair, I could click "Preview" and SyncToy would compare both folder pairs, before presenting me with a comprehensive list of operations it wanted to perform, to keep both pairs in sync.

![Synctoy's intuitive user interface](https://res.cloudinary.com/hibbard/image/upload/v1530007196/synctoy.png "Synctoy's intuitive user interface")

## So What About rsync?

Rsync is a command line program with a plethora of options. You can inspect these by entering `man rsync` at the terminal, or you can use it's GUI version which is [Grsync](http://pclosmag.com/html/Issues/200708/page04.html "A tutorial on using Grsync"). Both can be installed via `apt-get`.

I went the Grsync route and spent a good while reading up on the options. This is what I came up with to replicate SyncToy's echo:

-  Preserve time (-t)
-  Preserve owner (-o)
-  Preserve permissions (-p)
-  Preserve group (-g)
-  Delete on destination (-delete)
-  Do not leave filesystem (-x)
-  Verbose (-v)
-  Show transfer progress (-progress)
-  Always checksum (-c)
-  Copy symlinks as symlinks (-l)
-  Copy hardlinks as hardlinks (-H)
-  Show itemized changes list (-i)
-  Protect remote args (-s)
-  Recurse into directories (-r)

Under Linux my data partition and my external hard drive were mounted as `/mnt/files` and `/media/username/usb_disk_1` respectively.

Therefore the complete command to start my backup was:

```sh
rsync -t -o -p -g -x -v -c -l -H -i -s -r --delete --progress  /mnt/files /media/username/usb_disk_1
```

## First a Dry Run

I ran my backup with the additional `-n` parameter, which performs a dry run and makes no changes to the file system.

The output I got back was a touch overwhelming – reams and reams of information about every file rsync examined (maybe because both partitions are formatted with NTFS).

Anyway, as I have something like 175,000 files on my data partition, this was clearly unmanageable.

What I was really interested in was knowing whenever rsync renamed, deleted or created anything.

After a couple of tests, I concluded that in these cases, rsync's output looked like this:

```sh
Create new file:
>f+++++++++ files/my_test_file

Rename file:
*deleting files/my_test_file
>f+++++++++ files/my_new_test_file

Delete file:
*deleting files/my_test_file

Create new directory:
cd+++++++++ files/my_test_dir/

Rename directory:
*deleting files/my_test_dir/
cd+++++++++ files/my_test_directory/

Delete directory:
*deleting files/my_test_dir/
```

Now all I had to do is pipe rsync's output into `egrep` and look for these operations.

For good measure I could also save the output to a file, so that I could inspect the changes more thoroughly.-

### Dry run

```sh
rsync -t -o -p -g -x -v -c -l -H -i -s -r -n --delete --progress /mnt/files /media/username/usb_disk_1 | egrep "\*|>|cd\+" > dryrun
```

### Real thing

```sh
rsync -t -o -p -g -x -v -c -l -H -i -s -r --delete --progress /mnt/files /media/username/usb_disk_1 | egrep "\*|>|cd\+" > real_thing
```

### Conclusion

I made two batch files with the above commands which I run whenever I want to make a backup.

Now rysnc tells me exactly which changes it wants to make, which I can then examine before I have it make them. Happy day!

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.

I'd also be especially glad to hear if I am doing it wrong and there is an easier way to deal with incremental backups in Linux.
