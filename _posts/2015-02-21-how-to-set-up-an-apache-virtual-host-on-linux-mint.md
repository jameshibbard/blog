---
title: How to Set Up an Apache Virtual Host on Linux Mint
layout: post
permalink: /how-to-set-up-an-apache-virtual-host-on-linux-mint/
tags:
  - apache
  - linux
  - servers
excerpt_separator: <!--more-->
comments: false
---

Running Apache on my local machine helps me speed up my web development work. It means that I can use [root-relative urls](http://ifyoucodeittheywill.com/2009/03/absolute-relative-and-root-relative-urls/#root-relative-urls "Instead of being relative to where you are, root-relative is relative to where the root directory is."), server-side programming languages (such as PHP) and interface with a database — all without having to upload anything via FTP.

The only problem comes when you are working on multiple projects at the same time. If you create different directories for different projects within your web root (which defaults to `/var/www/html`), then the root-relative urls will break, as will any server-side includes you are using.

This is where [virtual hosts](http://httpd.apache.org/docs/2.2/vhosts/ "Apache Virtual Host documentation") come in. They allow you to create a separate domain for each of your projects, such as `http://project1/` and `http://project2/`.

<!--more-->

## Configuring Apache

I'm assuming you have Apache installed and configured. If you haven't you can grab the [LAMP server stack](http://en.wikipedia.org/wiki/LAMP_%28software_bundle%29 "LAMP — Linux, Apache, MySQL and PHP") with the following command in the terminal.

```sh
sudo apt-get install lamp-server^
```

You gotta love Linux, eh?

So, first off, let's make two directories for each of our projects:

```sh
mkdir /var/www/html/project1
mkdir /var/www/html/project2
```

And put a file in each:

```sh
echo "Project 1" > /var/www/html/project1/index.html
echo "Project 2" > /var/www/html/project2/index.html
```

Now, we'll need to create a virtual host file for each domain. We'll have to do this as root:

```sh
sudo touch /etc/apache2/sites-available/project1.conf
sudo touch /etc/apache2/sites-available/project2.conf
```

Open the first file in a text editor:

```sh
sudo gedit /etc/apache2/sites-available/project1.conf
```

and enter:

```apache
<VirtualHost *:80>
  ServerName project1
  DocumentRoot /var/www/html/project1
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Then open the second:

```sh
sudo gedit /etc/apache2/sites-available/project2.conf
```

and enter:

```apache
<VirtualHost *:80>
  ServerName project2
  DocumentRoot /var/www/html/project2
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Save these files and shut your editor.

Now we need to inform Apache that these sites exist. We can do this using the [a2ensite](http://man.he.net/man8/a2ensite "Man page for a2ensite and a2dissite commands") command:

```sh
sudo a2ensite project1.conf
sudo a2ensite project2.conf
```

Then restart Apache:

```sh
sudo service apache2 restart
```

The final step is to add these two new sites to your hosts file — a file used by your operating system to map hostnames to IP addresses.

```sh
sudo gedit /etc/hosts
```

Add the two new sites at the top of the file underneath localhost:

```apache
127.0.0.1  localhost
127.0.0.1 project1
127.0.0.1 project2
```

Now visit [http://project1/](http://project1/ "The mighty project 1") and [http://project2/](http://project2/ "The splendiferous project 2")  respectively. If all has gone well, you should see the files we created earlier.

**Note:** If you want to remove the sites, follow the steps in reverse and use `a2dissite`.
