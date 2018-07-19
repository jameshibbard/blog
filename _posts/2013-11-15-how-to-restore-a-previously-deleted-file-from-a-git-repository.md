---
title: How to Restore a Previously Deleted File from a Git Repository
layout: post
permalink: /how-to-restore-a-previously-deleted-file-from-a-git-repository/
tags:
  - git
  - working with files
excerpt_separator: <!--more-->
comments: false
---

I was recently working on a project and realised that I needed a file that I had long since deleted.

On the one hand, this wasn't very tragic as I had the project under version control using GIT. However, finding the exact command to restore a single file from a rather old commit proved to be a little more difficult, so I thought I'd make note of it here.

<!--more-->

## Tracking Deleted Files

Before getting into how to restore them, it's important to ensure that deleted files are actually being tracked in the first place. Let me elaborate:

When you initially set up version control for a project, it's common to see something along the lines of:

```sh
cd 'My Project'
git init
git add .
git commit -m 'Initial commit'
```

And although this is fine for the initial commit, where by definition every file is new, it's important to realise that `git add .` will only stage changed or new files (not deleted ones).

Alternatively, `git add -u` will stage changes to any altered or deleted files, but this is not always perfect, as it will ignore new files.

When you want to track changes to all files whether they be new, altered or deleted, it makes most sense to use `git add -A`, which is a handy shortcut for both of the above.

## Finding and Restoring the Deleted Files

Assuming that deleted files are being tracked, the next thing to do is to find out in which commit they were deleted.

```sh
git log --diff-filter=D --summary
```

Is a handy command to do just this. It will list all the commits which have deleted files, as well as the files deleted.

The output will look something like this:

```sh
commit fee78a9f30e35841ad5c0a6927661aeebdf368b6
Author: hibbard.eu <hibbard.eu@an-email-address.com>
Date: Fri Nov 15 13:43:30 2013 +0100
Deleted a very important file
delete mode 100644 project/important/file
```

Now you'll need to make a note of the commit id, then enter:

```sh
git checkout commit_id^1 file_name`
```

In my case this would have been:

```sh
git checkout fee78...368b6^1 project/important/file
```

And voilà, you have your file back!

Note: the commit id will reference the commit where the file was already deleted. By appending `^1` to it, you are referencing the commit before that.

### References

- [Difference between "git add -A" and "git add .""](http://stackoverflow.com/questions/572549/difference-between-git-add-a-and-git-add "The difference explained with examples on StackOverflow")
- [Restore a deleted file in a Git repo](http://stackoverflow.com/questions/953481/restore-a-deleted-file-in-a-git-repo "Various suggestions on StackOverflow")
