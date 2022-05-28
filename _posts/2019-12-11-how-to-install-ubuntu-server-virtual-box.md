---
title: How to Install Ubuntu Server on VirtualBox
layout: post
permalink: /install-ubuntu-virtual-box/
excerpt_separator: <!--more-->
tags:
  - servers
  - ssh
  - ubuntu
  - virtualbox
twitter:
  title: "How to Install Ubuntu Server on VirtualBox"
  description: "In this post I'll show you how to install Ubuntu Server 22.04 on VirtualBox. I'll also demonstrate how to connect to the Ubuntu instance via SSH."
  image_url: https://res.cloudinary.com/hibbard/image/upload/f_auto,w_800/v1536153775/stock/crayons.jpg
image_shadow: true
---

In this post I'll show you how to install Ubuntu Server 22.04 LTS (Jammy Jellyfish) on Oracle's VirtualBox. I'll also demonstrate how to connect to the Ubuntu instance via SSH, as well as how to run VirtualBox in headless mode.

Let's get started!

<!--more-->

---

**Note:** this post was updated on 28<sup>th</sup> May, 2022 to use the latest versions of Ubuntu Server and VirtualBox.

---

## What Is VirtualBox?

VirtualBox is a software virtualization package that you can install on your operating system (just as you would a normal program). It supports the creation and management of virtual machines into which you can install a second operating system.

In VirtualBox terminology, the operating system on which you install VirtualBox (i.e. your regular OS) is called the _host_. The operating system you install within VirtualBox (i.e. inside the virtual machine) is called the _guest_.

For this tutorial, I'll be using Linux Mint 20.3 as the host OS, but there's no reason you couldn't use a different Linux distro, or macOS, or Windows (if you're so inclined).

## Install VirtualBox

The first thing to do is to get VirtualBox installed. I'll not go into much detail here, as there are comprehensive instructions for all of the main operating systems [on the project's homepage](https://www.virtualbox.org/wiki/Downloads).

Personally, I downloaded and installed the deb package for Ubuntu 20.04. This is because the VirtualBox version in the Mint repos is slightly outdated and I wanted to be running the latest version.

## Download Ubuntu Server

The next thing to do is to grab a copy of Ubuntu Server. You can do this from their [download page](https://www.ubuntu.com/download/server). Select option 2 (_Manual server installation_) which will download a 1.37GB ISO file to your PC.

At the time of writing the current LTS version is Ubuntu Server 22.04 and this is what I'll be using. It's supported until April 2025 and is available 64-bit.

## Create a New Virtual Machine

Start up VirtualBox. This should open the VirtualBox Manager, the interface from which you will administer all of your virtual machines.

![Welcome to VirtualBox](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-1.png)

Next Click on _New_ (in the top right of the VirtualBox Manager), give your virtual machine a name and the two drop down menus should automatically update.

![Name and operating system](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-2.png)

Click _Next_. The wizard will now ask you to select the amount of memory (RAM) in megabytes to be allocated to the virtual machine. I chose 2GB (2048 megabytes).

![Memory size](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-3.png)

Click _Next_ and you will be prompted to add a virtual hard disk to the new machine. Make sure that _Create a virtual hard disk now_ is selected, then press _Create_.

![Hard disk](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-4.png)

Now we need to choose the file type for the new virtual hard disk. Make sure that _VDI (VirtualBox Disk Image)_ is checked and press _Next_.

![Hard disk file type](https://res.cloudinary.com/hibbard/image/upload/v1576058634/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-5.png)

On the next screen you will be asked whether the new virtual hard disk should grow as it is used (dynamically allocated) or if it should be created at its maximum size. Make sure that _dynamically allocated_ is selected, then click _Next_.

![Storage on physical hard disk](https://res.cloudinary.com/hibbard/image/upload/v1576058634/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-6.png)

Finally, select the size of the virtual hard disk in megabytes. The default size of 10GB should be plenty, but feel free to increase this as you see fit. Then click _Create_.

![File location and size](https://res.cloudinary.com/hibbard/image/upload/v1576058634/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-7.png)

The hard disk should now be created and you should find yourself back in the VirtualBox Manager. You should be able to see your newly created virtual machine listed on the left.

## Install Ubuntu Server in the Virtual Machine

Make sure your virtual machine is selected and press _Start_. VirtualBox Manager will ask you to select a virtual optical disk file or a physical optical drive to start the virtual machine from. Click on the folder with the upwards arrow on the right side of the dialogue, select the ISO file you downloaded previously and press _Start_.

![Select start-up disk](https://res.cloudinary.com/hibbard/image/upload/v1576058634/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-8.png)

The Ubuntu installation process will now begin. It consists of multiple steps and is quite painless.

### The Welcome Screen

Here you should select your preferred language. I'm using English.

![The Welcome Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-1.png){:class="terminal"}

### Installer Update

If an installer update is available, you can choose to install it or ignore it. I chose to install the update.

![The Installer Update Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-2.png){:class="terminal"}

This will download the update, then restart the installer.

### Keyboard Configuration

Here you should select a keyboard layout. As I'm using a German keyboard, I asked Ubuntu to detect my layout, which it did with a couple of simple questions.

![The Keyboard Configuration Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-3.png){:class="terminal"}

### Type of Install

Next you should choose between the default install that contains a curated set of packages and a minimized version, which has been customized to have a small runtime footprint. Choose the default option.

![The Type of Install Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-4.png){:class="terminal"}

### Network Connections

Here Ubuntu will attempt to configure the standard network interface. Normally you can just accept the default and select _Done_.

![The Network Connections Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-5.png){:class="terminal"}

### Configure Proxy

If your system requires a proxy to connect to the internet (mine doesn't), enter its details in the next dialogue. Then select _Done_.

![The Configure Proxy Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-6.png){:class="terminal"}

### Configure Ubuntu Archive Mirror

If you wish to use an alternative mirror for Ubuntu, you can enter the details here. Otherwise accept the default mirror by selecting _Done_. I accepted the default.

![The Ubuntu Archive Mirror Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-7.png){:class="terminal"}

### Guided Storage Configuration

The installer can guide you through partitioning an entire disk or, if you prefer, you can do it manually. If you choose to partition an entire disk you will still have a chance to review and modify the results before Ubuntu is installed. I selected _Use An Entire Disk_.

You can optionally instruct the installer to set up the disk as an LVM group, as well as to encrypt it using LUKS. I chose to go with the LVM setup, as LVM offers a number of benifits, such as allowing easier backups of a running server. You can read more about LVM here: [What is LVM and what is it used for?](https://askubuntu.com/q/3596/645148)

![The Guided Storage Configuration Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-8.png){:class="terminal"}

### Storage Configuration Summary

The next screen summarizes the choices you made in the previous step. If you are happy with everything, select _Done_.

![The Storage Configuration Summary](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-9.png){:class="terminal"}

As this is a "destructive action". I was asked to confirm my choice with _Continue_.

![The Confirm Destructive Action Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058632/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-10.png){:class="terminal"}

### Profile Setup

Here you are required to enter:

  - Your (real) name
  - Your server's name
  - Your username
  - Password

Fill these details out as you see fit.

![The Profile Setup Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-11.png){:class="terminal"}

### SSH Setup

Here we have a chance to install the [OpenSSH server package](https://ubuntu.com/server/docs/service-openssh). We'll need this to connect to the virtual machine via SSH later on, so ensure that you select it.

You also have the opportunity to import your SSH keys from GitHub or Launchpad. I selected _No_ for this option.

![The SSH Setup Screen](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-12.png){:class="terminal"}

### Featured Server Snaps

Here you can select from a list of popular snaps to install on your system. [Snaps](https://tutorials.ubuntu.com/tutorial/basic-snap-usage#0) are self-contained  software packages that work across a range of Linux distributions. I didn't select any.

![The Featured Server Snaps screen](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-13.png){:class="terminal"}

### Install and reboot

And that's it, the installaler will now install Ubuntu 22.04. Once it is finished you should select _Reboot Now_

![The Reboot screen](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/ubuntu-install-14.png){:class="terminal"}

Ubuntu will ask you to remove the installation medium and press <kbd>Enter</kbd>. You can remove the disk via_Devices_ > _Optical Drives_ > _Remove disk from virtual drive_. You will need to put a check mark next to _ubuntu-22.04-live-server-amd64.iso_ if it is not selected already.

## Up and Running with SSH

Once your virtual machine has rebooted and you have logged in, you'll probably notice that some packages can be updated.

Let's fix that:

```sh
sudo apt update
sudo apt upgrade
```

If you see a _Daemons using outdated libraries_ dialogue asking you which services should be restarted, just accept the defaults and navigate to _OK_ by pressing <kbd>TAB</kbd>.

Now let's double check that SSH is installed (it should be if you selected the option _Install OpenSSH server_ during instalation).

```sh
jim@odin:~$ ssh

usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-E log_file] [-e escape_char]
           [-F configfile] [-I pkcs11] [-i identity_file]
           [-J [user@]host[:port]] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-p port] [-Q query_option] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           [user@]hostname [command]
```

If you get a "command not found" error, you can install it with:

```
sudo apt-get install openssh-server
```

The next step is to give our Ubuntu server an IP address on our local network. To do this, power off the virtual machine using `sudo poweroff` or _Machine_ > _ACPI Shutdown_.

Then, in VirtualBox Manager, make sure your machine is selected and click _Settings_. Click on _Network_ on the left and change the setting _Adapter 1_ > NAT_ to _"_Bridged Adapter_"_ and click _OK_.

![VirtualBox Manager - Network Adapter 1](https://res.cloudinary.com/hibbard/image/upload/v1576058634/ubuntu-22.04-virtualbox-6.1.32/virtualbox-manager-9.png)

Start up your virtual machine, then enter `ip address` (in the guest) and note the IP address assigned to your main network adapter. In my case this was `192.168.178.40`.

**Note:** it is also possible to stick with the original NAT interface and SSH into the guest using port forwarding. You can read more about that [here](https://stackoverflow.com/a/10532299/1136887). You can find information on all of the VBox network settings [in this comprehensive guide](https://www.nakivo.com/blog/virtualbox-network-setting-guide/).

## Starting and Stopping VirtualBox in Headless Mode

You might have noticed, working with the VirtualBox Manager and the guest OS is a bit of a pain. If you're going to continue doing this, you should at least [install the guest additions](https://www.virtualbox.org/manual/ch04.html), as well as enable clipboard support.

There is a slightly nicer way however — you can start and stop the virtual machine using the [VBoxManage command](https://www.virtualbox.org/manual/ch08.html) from your terminal.

To power on:

```sh
VBoxManage startvm "Ubuntu Server 22.04" --type headless
```

And to power off:
```sh
VBoxManage controlvm "Ubuntu Server 22.04" poweroff
```

Where "Ubuntu Server 22.04" is whatever you called your virtual machine (the name it has in the VirtualBox Manager GUI).

## Connecting to the Ubuntu Server

Let's go ahead and start the Ubuntu server in headless mode, before connecting to it via SSH.

> Note: The following commands should be run on your host.

On most *nix systems, the SSH client software should be part of the default installation. If you don't have it available, you should be able to grab it from the repos, like so:

```sh
sudo apt install openssh-client
```

or just hit DuckDuckGo.

Then (ensuring that you replace "jim" and the IP address with your corresponding values) you can connect like so:

```sh
ssh jim@192.168.178.40
```

This will give you a warning that the host's authenticity cannot be established and ask you if you want to continue connecting. Answer "yes".

Next, it will prompt you for your password. Enter it and you will be connected to your Ubuntu server from your host OS.

![Connected to the Ubuntu server via SSH](https://res.cloudinary.com/hibbard/image/upload/v1576058634/ubuntu-22.04-virtualbox-6.1.32/ubuntu-ssh-1.png){:class="terminal"}

### For Windows Users

If you're running Windows you'll need to install a [SSH client such as PuTTY](https://www.ssh.com/ssh/putty/download#sec-Download-PuTTY-installation-package-for-Windows).

When PuTTY starts, a window titled _PuTTY Configuration_ should open. This window has a configuration pane on the left, a _Host Name_ field and other options in the middle, and a pane for saving session profiles in the lower right area.

For simple use, all you need to do is to enter the IP address of the host you want to connect to in the _Host Name_ field and click _Open_.

![PuTTY Configuration](https://res.cloudinary.com/hibbard/image/upload/v1576058633/ubuntu-22.04-virtualbox-6.1.32/ubuntu-putty-1.png)

## Generate and Install a SSH Key Pair

SSH keys offer a secure manner of logging into a server without the need of a password.

In a nutshell, this depends upon you generating a public and a private SSH key pair. The private key is kept on your PC (and should be guarded carefully). The public key is copied over to the server you wish to connect to.

SSH keys are a complex subject and as such, out of the scope of this tutorial. If you'd like to find out more, I recommend looking for a dedicated tutorial ([such as this one](https://www.howtoforge.com/linux-basics-how-to-install-ssh-keys-on-the-shell)).

### Generate the Keys

On *nix systems (Windows users see the next section), you can generate your key pair with the following command:

```sh
ssh-keygen -o -b 4096 -t rsa
```

The `-o` option instructs ssh-keygen to store the private key in the new OpenSSH format instead of the old (and more compatible PEM format). This is advisable, as the new OpenSSH format has an increased resistance to brute-force password cracking.

The `-b` option is used to set the key length to 4096 bits instead of the default 1024 bits for security reasons.

In the following dialogue you will be required to answer a couple of questions:

  - Where to save the newly generated key pair
  - Which passphrase to use

Here you can accept the default location and leave the passphrase blank by pressing <kbd>Return</kbd>.

ssh-keygen will then output a summary of what it has done:

```sh
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jim/.ssh/id_rsa):
Created directory '/home/jim/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/jim/.ssh/id_rsa.
Your public key has been saved in /home/jim/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:sx5uJeVdH/cT/1+GxsSWYzmjf5hUaE33f/e57EbqBfY jim@fitz
The key's randomart image is:
+---[RSA 4096]----+
|                 |
|                o|
|               +o|
|          .  .+==|
|        So . =@oB|
|        .oo o*+BB|
|        oo  ..*EX|
|       o..   +=+=|
|       .o   ..+=+|
+----[SHA256]-----+
```

### Copy the Public Key to the Ubuntu Server

To copy the public key to the Ubuntu server use:

```sh
ssh-copy-id -i ~/.ssh/id_rsa.pub jim@192.168.178.40
```

Where `~/.ssh/id_rsa.pub` is the path to your public key, taken from the output above. And where `jim@192.168.178.40` should be altered to reflect your details.

The command will run and you should be asked for your server password. Enter it, then attempt to log into the server like so:

```sh
ssh jim@192.168.178.40
```

This time you should be in without a password.

## For Windows Users

You should be able to use a tool like [PuTTYgen](https://www.ssh.com/ssh/putty/windows/puttygen
) to achieve the same thing. Here is a tutorial on using PuTTYgen to [create a new key pair for authentication](https://www.ssh.com/ssh/putty/windows/puttygen#sec-Creating-a-new-key-pair-for-authentication).

You will have a little more leg work when it comes to copying the key to the server, where you will need to add the public key to a `~/.ssh/authorized_keys` file.

You can do that like so (on the guest):

```
cd
mkdir .ssh
cd .ssh
nano authorized_keys
```

This will create the appropriate file, then open the nano editor into which you can copy your newly generated public key.

When you're done, press <kbd>Ctrl</kbd> + <kbd>X</kbd> to save your changes and exit nano.

## Conclusion

This has been quite a long post, but by the end of it you should have a working installation of Ubuntu Server running on VirtualBox that you can connect to from your host operating system via SSH.

If you have any questions or feedback, I'd be glad to hear your from you in the comments.
