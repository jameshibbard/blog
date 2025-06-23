---
title: Erase Sensitive Files from Git History with git-filter-repo
layout: post
permalink: /erase-sensitive-files-git-history-filter-repo/
tags:
  - git
  - secrets
  - security
  - repositories
  - tools
excerpt_separator: <!--more-->
twitter:
  title: "Erase Sensitive Files from Git History with git-filter-repo"
  description: "Accidentally committed secrets to Git? Learn how to remove sensitive files from your repo history using git filter-repo."
  image_url: https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit,w_800/v1750685390/git-filter-repo-header_at1vnh.jpg
image_shadow: true
---

![](https://res.cloudinary.com/hibbard/image/upload/v1750685390/git-filter-repo-header_at1vnh.jpg){: .header-image}

Ever accidentally commit a password to Git? You’re not the only one.

Last weekend, while working on a Rails project, I accidentally added my `database.yml` file to the repo and pushed it to GitHub. It contained my local MySQL password. No public exposure thankfully, but it still nagged at me. I didn’t want that file, or its contents, lingering in my Git history.

<!--more-->

After a bit of research, I found a quick and effective way to scrub it from history: a tool called [git filter-repo](https://github.com/newren/git-filter-repo). In this post, I’ll show you how to use it to remove committed secrets, credentials, or just a file that shouldn’t be there.

## Why `git rm` or Reverting a Commit Isn't Enough

Your first thought might be to delete the file with `git rm` or roll back the commit. But that won’t erase the damage.

Git is built to remember everything. Even if you remove the file from the latest commit, it remains in earlier ones. Anyone with access to your repo can still retrieve it—or see it in the commit history.

That’s not just inconvenient. It’s a security risk. Even if the credentials are no longer valid, their presence in your history could expose internal architecture or violate compliance policies.

To get rid of it for good, you need to rewrite history.

## Meet `git filter-repo`: A Cleaner Solution

`git filter-repo` is a fast, flexible, and actively maintained tool for rewriting Git history. It lets you permanently remove files, directories, commits, authors—anything you want to scrub from your repository’s timeline.

It’s also the [officially recommended replacement](https://git-scm.com/docs/git-filter-branch#_warning) for `git filter-branch`, the older built-in command that’s now widely considered obsolete due to its slowness and complexity.

You might’ve also come across [`bfg-repo-cleaner`](https://github.com/rtyley/bfg-repo-cleaner), which is still a valid option. It’s reliable and works well for simple tasks like deleting large files or purging secrets. But it does come with a few caveats ([as detailed here](https://github.com/newren/git-filter-repo?tab=readme-ov-file#bfg-repo-cleaner)) and it runs on Java, so you'll need that installed on your system.

### Installation

If you're on Ubuntu or any Debian-based system, the easiest way to install `git-filter-repo` is via `apt`:

```bash
sudo apt install git-filter-repo
```

For macOS users, Homebrew has you covered:

```bash
brew install git-filter-repo
```

Alternatively, `git-filter-repo` is available as a Python package via PyPI, which works cross-platform:

```bash
pip install git-filter-repo
```

You can also install it manually by downloading the [script directly](https://raw.githubusercontent.com/newren/git-filter-repo/main/git-filter-repo), placing it somewhere in your `$PATH`, and ensuring it's executable.

For other methods—including installing from source, using a Makefile, or additional Windows-specific instructions—refer to the official [INSTALL.md](https://github.com/newren/git-filter-repo/blob/main/INSTALL.md) on GitHub.

Absolutely, My Lord. Here’s the revised section—clearer, more explanatory, and grounded in what each step actually does, with a proper link back to the official recommendation:

## Step-by-Step: Removing a File from Git History

With `git filter-repo` installed, here’s how to permanently remove a file from your Git history—such as my accidentally committed `config/database.yml`.

There are two main approaches: the **canonical method**, recommended by the tool’s author, and a **pragmatic shortcut** that works fine if you’re confident and know the risks.

> ⚠️ **Warning:** This process rewrites Git history and cannot be undone. Make a backup before you begin. Clone the repo elsewhere, zip up your project folder—anything. I’m not responsible if you wipe out your repository. Proceed with caution.

### Option 1: The Canonical Way (Fresh Clone)

This is the approach recommended by the tool’s [official documentation](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#FRESHCLONE), and it’s the safest way to proceed.

#### 1. Create a fresh mirror of your repository

```bash
git clone --mirror https://github.com/your-user/your-repo.git
cd your-repo.git
```

This creates a bare clone with all branches, tags, and references—essentially the full repo history—without any working files. It’s the clean slate `git filter-repo` expects.

#### 2. Run the filter to delete the file

```bash
git filter-repo --path config/database.yml --invert-paths
```

This tells `git filter-repo` to *remove* (`--invert-paths`) every instance of the file from all history. See [Filtering based on paths](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#_filtering_based_on_paths_see_also_filename_callback) in user manual for more details

#### 3. Push the cleaned history

```bash
git push --force
```

Force-push is required because the rewritten history no longer matches the remote.

---

### Option 2: The Pragmatic Shortcut (Local Repo)

If you don’t want to clone again and are okay rewriting history in place:

```bash
git filter-repo --path config/database.yml --invert-paths --force
git push --force
```

This does exactly the same thing, but runs against your current working repository. The `--force` flag is required because `git filter-repo` will otherwise abort when it detects that your repo isn’t freshly cloned.

By taking this route, you bypass the safety net. Your `.git` directory will be rewritten in place. Any mistakes, and you may not be able to recover unless you’ve backed it up or can reclone it. So proceed only if you’re confident you’ve got a clean working state—or don’t mind burning it all down.

Either way, the result is the same: the file is scrubbed from every commit it ever touched. It’s no longer in your Git history. Not even in the shadows.

## What to Do Next: Rotate Keys and Notify Your Team

Even though the file is now gone from Git history, its contents may have been exposed during the time it was accessible. If it contained any secrets—like API keys, database passwords, or tokens—you should treat them as compromised and rotate them. Especially if the exposed data could grant access to anything internet-facing, don’t take chances. You don’t necessarily need to rotate everything, but rotate anything that could plausibly be used externally. Better to be overly cautious than to discover abuse later.

It’s also wise to review your access logs. If the repository was public or shared with others during the time the file was present, see if there were any unexpected clones or forks. Even if it’s unlikely that someone accessed the secret, assuming the worst puts you on safer ground.

Let your team know what happened. Anyone who cloned or forked the repository before the cleanup will still have a copy of the secret in their local history. Ask them to delete their clone or re-clone the repository entirely. You’ll want everyone working from the same cleaned state moving forward.

## Avoiding This in the Future

The best defense against this kind of thing is a proactive setup that makes it harder to slip up again.

First, use a properly configured [`.gitignore`](https://git-scm.com/docs/gitignore) file to keep sensitive files—like `config/database.yml`, `.env`, or credential stores—out of version control from the start. This is your first and simplest safeguard.

Instead of hardcoding secrets, store them in [environment variables](https://12factor.net/config) or use a dedicated secrets manager which let you separate configuration from code.

You can also set up [Git pre-commit hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) to block bad commits. Frameworks like [`pre-commit`](https://pre-commit.com/) make this easy to manage across projects, letting you run linters, formatters, or custom scripts automatically before each commit.

For secret detection specifically, [`git-secrets`](https://github.com/awslabs/git-secrets) scans your changes for known patterns (like AWS keys) and aborts the commit if something leaks through.

## Conclusion: Clean Repos, Clean Conscience

In the end, scrubbing my `database.yml` from history was quick and painless and `git filter-repo` did exactly what it promised.

But if you’ve ever exposed something critical—an API key, a password, anything live, you’ll know the sinking feeling. The good news? With the right tools, you can clean up the mess and move on with confidence.

I hope this post helped—if you have any questions or thoughts, feel free to drop them in the comments below.
