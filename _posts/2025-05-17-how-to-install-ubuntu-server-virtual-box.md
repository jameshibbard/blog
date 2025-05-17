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
  description: "In this post I'll show you how to install Ubuntu Server 24.04 on VirtualBox. I'll also demonstrate how to connect to the Ubuntu instance via SSH."
  image_url: https://res.cloudinary.com/hibbard/image/upload/f_auto,w_800/v1536153775/stock/crayons.jpg
image_shadow: true
---

In this post I'll show you how to install Ubuntu Server 24.04 LTS (Jammy Jellyfish) on Oracle's VirtualBox. I'll also demonstrate how to connect to the Ubuntu instance via SSH, as well as how to run VirtualBox in headless mode.

Let's get started!

<!--more-->

---

**Note:** this post was updated on 17<sup>th</sup> May, 2025 to use the latest versions of Ubuntu Server and VirtualBox.

---

## What Is VirtualBox?

VirtualBox is a software virtualization package that you can install on your operating system (just as you would a normal program). It supports the creation and management of virtual machines into which you can install a second operating system.

In VirtualBox terminology, the operating system on which you install VirtualBox (i.e. your regular OS) is called the _host_. The operating system you install within VirtualBox (i.e. inside the virtual machine) is called the _guest_.

For this tutorial, I'll be using Linux Mint 22.1 as the host OS, but there's no reason you couldn't use a different Linux distro, or macOS, or Windows (if you're so inclined).

## Install VirtualBox

The first thing to do is to get VirtualBox installed. The process varies slightly depending on your host system, but Oracle provides platform-specific installers that are straightforward to use. If you encounter issues or have special requirements, consult the [User Guide](https://www.virtualbox.org/manual/), especially these sections:
- [Installing Oracle VirtualBox and Extension Packs](https://www.virtualbox.org/manual/topics/Introduction.html#intro-installing)
- [Installation Details](https://www.virtualbox.org/manual/topics/installation.html#installation)

On most Linux distributions, you can install VirtualBox directly via your package manager or software center. For example, on Linux Mint, simply searching for "VirtualBox" will pull in the appropriate package from the official repositories. This is what I did, and it installed version 7.0.16 — slightly behind the latest version (7.1.8) available from the [VirtualBox website](https://www.virtualbox.org/).

## Download Ubuntu Server

The next thing to do is to grab a copy of Ubuntu Server. You can do this from their [download page](https://www.ubuntu.com/download/server). Hit the green download button and this will download a 3.2GB ISO file to your PC.

At the time of writing the current LTS version is Ubuntu Server 24.04 and this is what I'll be using. It's supported until April 2029.

## Create a New Virtual Machine

Start up VirtualBox. This should open the VirtualBox Manager, the interface from which you will administer all of your virtual machines.

![VirtualBox Manager welcome screen](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747483043/ubuntu-24.04.04-virtualbox-7.0.16/virtualbox-manager-1.png)

Click on _New_ (in the top right of the VirtualBox Manager) to proceed.

![Virtual machine name and OS selection screen](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747487367/ubuntu-24.04.04-virtualbox-7.0.16/virtualbox-manager-2.png)

Give your virtual machine a name — the two dropdown menus should update automatically based on your input.

Next, select the ISO image. To do this, click the _ISO Image_ dropdown (which initially shows _&lt;not selected&gt;_), then choose _Other..._. A file picker will open. Navigate to the file you downloaded earlier — it should be called `ubuntu-24.04.2-live-server-amd64.iso`. Select it and confirm (e.g. click _Open_).

Also, tick the box labeled _Skip Unattended Installation_. This gives you full visibility into the Ubuntu setup process and allows for more fine-grained control over how the system is installed.

Once done, click _Next_.

![Hardware configuration screen: RAM and CPU](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747483424/ubuntu-24.04.04-virtualbox-7.0.16/virtualbox-manager-3.png)

The wizard will now prompt you to choose how much memory (RAM), in megabytes, to allocate to the virtual machine. I went with 2GB (2048MB).

You'll also be asked to select the number of processor cores to assign — I chose 2.

Having made your selection, click _Next_.

![Virtual hard disk creation screen](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747483333/ubuntu-24.04.04-virtualbox-7.0.16/virtualbox-manager-4.png)

On the next screen you’ll be prompted to add a virtual hard disk to the new machine. Make sure _Create a Virtual Hard Disk Now_ is selected.

On this same screen, you can also set the size of the virtual hard disk — I left it at the default 25GB. If you leave the _Pre-allocate Full Size_ box unchecked, the disk will grow dynamically as needed rather than occupying the full 25GB immediately.

Once you’ve configured everything, click _Next_ to continue.

![Summary screen showing VM configuration](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747487368/ubuntu-24.04.04-virtualbox-7.0.16/virtualbox-manager-5.png)

Finally, you'll see a summary of the configuration you have chosen. Assuming everything is correct, click _Finish_.

The hard disk should now be created and you should find yourself back in the VirtualBox Manager. You should see your newly created virtual machine listed on the left.

## Install Ubuntu Server in the Virtual Machine

In this section, we’ll walk through installing Ubuntu Server as a **guest** operating system inside VirtualBox, running on your **host** machine.

---

Before we get started, a quick note for those of you using a 4K monitor: the VirtualBox window will likely appear incredibly small, making it nearly impossible to read anything. To fix this, open VirtualBox, go to _File > Preferences > Display_, and set the _Scale Factor_ to _200%_ (or whatever looks best on your screen). This setting will make the VirtualBox interface much more usable.

---

Now, with your virtual machine selected, press _Start_. Once the machine boots, you’ll see a screen offering the option to _Try or Install Ubuntu Server_. The first option — _Install Ubuntu Server_ — will already be selected. Click anywhere inside the VM window to focus it.

Note the first time you click inside the VM, VirtualBox will display a message explaining that you've clicked into the virtual machine and that it is capturing your keyboard and mouse input. This is normal. Simply tick the box that says _Do not show this message again_, then click _Capture_.

![VirtualBox information dialog explaining mouse and keyboard capture behavior when interacting with the virtual machine window.](https://res.cloudinary.com/hibbard/image/upload/w_600,dpr_auto,q_auto/v1747493397/ubuntu-24.04.04-virtualbox-7.0.16/capture-dialogue.png)

You will also notice a semi-transparent sidebar on the right side of the VM window. This sidebar cannot be permanently disabled ([source](https://forums.virtualbox.org/viewtopic.php?t=111390)). However, you can manage it in two ways:
- Click the speech bubble icon with an exclamation mark at the top of the sidebar to temporarily hide it.
- For each message, click the speech bubble with a line through it to permanently suppress that specific message.

![Try or Install Ubuntu Server GRUB boot menu](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747492951/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-01.png)

Once you're done dismissing or ignoring those messages, press <kbd>Return</kbd> (or wait a few seconds) to move into the Ubuntu welcome screen. From there, the installation process will begin.

### Navigating the Install Screen

Once the Ubuntu installer loads, you'll interact with it entirely via the keyboard.

Use <kbd>Tab</kbd> to move forward through the available options, and <kbd>Shift</kbd>+<kbd>Tab</kbd> to move backwards. Press <kbd>Space</kbd> to select or deselect checkboxes, and <kbd>Return</kbd> to confirm a choice or activate a highlighted button.

These keys are all you'll need to navigate and complete the installation process in the terminal-based interface. You'll first use them on the welcome screen to select your language and begin the setup.

### The Welcome Screen

Here you should select your preferred language. I'm using English.

![Language selection screen in Ubuntu Server installer](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495008/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-02.png)

### Keyboard Configuration

Here you should select a keyboard layout. As I'm using a German keyboard, I asked Ubuntu to detect my layout, which it did with a couple of simple questions.

![Keyboard layout selection screen with English (US) selected](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495009/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-03.png)

### Type of Install

Next you should choose between the default install that contains a curated set of packages and a minimized version, which has been customized to have a small runtime footprint. Choose the default option.

![Installation type screen showing default and minimized Ubuntu Server options](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495010/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-04.png)

The _Search for third-party drivers_ option scans your system for hardware that may benefit from proprietary drivers (e.g. graphics, Wi-Fi, or input devices) and installs them if available. I left it unchecked.

### Network Connections

Here Ubuntu will attempt to configure the standard network interface. Normally you can just accept the default and select _Done_.

![Network configuration screen with a DHCP-assigned IP address](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495011/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-05.png)

### Configure Proxy

If your system requires a proxy to connect to the internet (mine doesn't), enter its details in the next dialogue. Then select _Done_.

![Proxy configuration screen with empty address field](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495013/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-06.png)

### Configure Ubuntu Archive Mirror

If you wish to use an alternative mirror for Ubuntu, you can enter the details here. Otherwise accept the default mirror by selecting _Done_. I accepted the default.

![Ubuntu mirror configuration screen using Greek archive server](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495015/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-07.png)

### Guided Storage Configuration

The installer can guide you through partitioning an entire disk or, if you prefer, you can do it manually. If you choose to partition an entire disk you will still have a chance to review and modify the results before Ubuntu is installed. I selected _Use an entire disk_.

You can optionally instruct the installer to set up the disk as an LVM group, as well as to encrypt it using LUKS. I chose to go with the LVM setup, as LVM offers a number of benifits, such as allowing easier backups of a running server. You can read more about LVM here: [What is LVM and what is it used for?](https://askubuntu.com/q/3596/645148)

![Guided storage configuration with entire disk and LVM group selected](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495017/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-08.png)

### Storage Configuration Summary

The next screen summarizes the choices you made in the previous step. If you are happy with everything, select _Done_.

![Storage configuration summary showing mount points and used devices](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495018/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-09.png)

As this is a "destructive action". I was asked to confirm my choice with _Continue_.

![Confirmation prompt for starting disk formatting and installation](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495020/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-10.png)

### Profile Setup

Here you are required to enter:

  - Your (real) name
  - Your server's name
  - Your username
  - Password

Fill these details out as you see fit.

![Profile setup screen with name, hostname, username, and password fields](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495022/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-11.png)

### Upgrade to Ubuntu Pro

At this point, you're given the option to enable **Ubuntu Pro**, a free subscription for personal use that extends security updates to a broader set of packages and adds support for various compliance standards.

![Ubuntu Pro upgrade prompt with "Skip for now" selected](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495024/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-12.png)

I chose to skip this step. You can always enable Ubuntu Pro later using the `pro attach` command.

### SSH Setup

Here we have a chance to install the [OpenSSH server package](https://ubuntu.com/server/docs/service-openssh). We'll need this to connect to the virtual machine via SSH later on, so ensure that you select it.

You also have the opportunity to import your SSH keys from GitHub or Launchpad. I selected _No_ for this option.

![SSH configuration screen with OpenSSH server and password auth enabled](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495026/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-13.png)

### Featured Server Snaps

Here you can select from a list of popular snaps to install on your system. [Snaps](https://tutorials.ubuntu.com/tutorial/basic-snap-usage#0) are self-contained  software packages that work across a range of Linux distributions. I didn't select any.

![Featured server snaps selection screen with no snaps selected](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495027/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-14.png)

### Install and reboot

And that's it, the installaler will now install Ubuntu 24.04. Once it is finished you should select _Reboot Now_

![Installation complete screen with final log and "Reboot Now" option](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747495029/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-install-15.png)

Ubuntu will ask you to remove the installation medium and press <kbd>Enter</kbd>. You can remove the disk via _Devices_ > _Optical Drives_ > _Remove disk from virtual drive_. You will need to put a check mark next to _ubuntu-22.04-live-server-amd64.iso_ if it is not selected already.

If for some reason this option is grayed out (it was for me), just hit <kbd>Return</kbd>

## Up and Running with SSH

Once your virtual machine has rebooted and you have logged in, you'll probably notice that some packages can be updated.

Let's fix that:

```sh
sudo apt update
sudo apt upgrade
```

Now let's double check that SSH is installed (it should be if you selected the option _Install OpenSSH server_ during instalation).

```sh
jim@grunt:~$ ssh
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface] [-b bind_address]
           [-c cipher_spec] [-D [bind_address:]port] [-E log_file]
           [-e escape_char] [-F configfile] [-I pkcs11] [-i identity_file]
           [-J destination] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-P tag] [-p port] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           destination [command [argument ...]]
       ssh [-Q query_option]
```

If you get a "command not found" error, you can install it with:

```
sudo apt-get install openssh-server
```

The next step is to give our Ubuntu server an IP address on our local network. To do this, power off the virtual machine using `sudo poweroff` or _Machine_ > _ACPI Shutdown_.

Then, in VirtualBox Manager, make sure your machine is selected and click _Settings_. Click on _Network_ on the left and change the setting _Adapter 1_ > _NAT_ to _Bridged Adapter_ and click _OK_.

![VirtualBox network settings showing Adapter 1 configured as a bridged adapter using enp3s0.](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747498239/ubuntu-24.04.04-virtualbox-7.0.16/virtualbox-manager-6.png)

Start up your virtual machine, then enter `ip address` (in the guest) and note the IP address assigned to your main network adapter. In my case this was `192.168.178.49`.

**Note:** it is also possible to stick with the original NAT interface and SSH into the guest using port forwarding. You can read more about that [here](https://stackoverflow.com/a/10532299/1136887). You can find information on all of the VBox network settings [in this comprehensive guide](https://www.nakivo.com/blog/virtualbox-network-setting-guide/).

## Starting and Stopping VirtualBox in Headless Mode

You might have noticed, working with the VirtualBox Manager and the guest OS is a bit of a pain. If you're going to continue doing this, you should at least [install the guest additions](https://www.virtualbox.org/manual/ch04.html), as well as enable [clipboard support](https://windowsloop.com/virtualbox-clipboard-sharing/).

There is a slightly nicer way however — you can start and stop the virtual machine using the [VBoxManage command](https://www.virtualbox.org/manual/ch08.html) from your terminal.

To power on:

```sh
VBoxManage startvm "Ubuntu Server 24.04" --type headless
```

And to power off:
```sh
VBoxManage controlvm "Ubuntu Server 24.04" poweroff
```

Where "Ubuntu Server 24.04" is whatever you called your virtual machine (the name it has in the VirtualBox Manager GUI).

## Connecting to the Ubuntu Server

First, make sure the virtual machine is shut down. If it's still running, open the terminal inside Ubuntu and run `sudo poweroff`.

Once it's powered down, we’ll start it again in headless mode:

```sh
VBoxManage startvm "Ubuntu Server 24.04" --type headless
```

And then connect to it via SSH.

> Note: The following commands should be run on your host.

On most *nix systems, the SSH client software should be part of the default installation. If you don't have it available, you should be able to grab it from the repos, like so:

```sh
sudo apt install openssh-client
```

or just hit [DuckDuckGo](https://duckduckgo.com/?q=install%20ssh%20linux&t=lm&ia=web).

Then (ensuring that you replace "jim" and the IP address with your corresponding values) you can connect like so:

```sh
ssh jim@192.168.178.49
```

This will give you a warning that the host's authenticity cannot be established and ask you if you want to continue connecting. Answer "yes".

Next, it will prompt you for your password. Enter it and you will be connected to your Ubuntu server from your host OS.

![Terminal showing a successful SSH login to the Ubuntu Server from the host machine — yay!](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1747498441/ubuntu-24.04.04-virtualbox-7.0.16/ubuntu-ssh-1.png)

### For Windows Users

If you're running Windows you'll need to install a [SSH client such as PuTTY](https://www.ssh.com/academy/ssh/putty/download#sec-Download-PuTTY-installation-package-for-Windows).

When PuTTY starts, a window titled _PuTTY Configuration_ should open. This window has a configuration pane on the left, a _Host Name_ field and other options in the middle, and a pane for saving session profiles in the lower right area.

For simple use, all you need to do is to enter the IP address of the host you want to connect to in the _Host Name_ field and click _Open_.

![PuTTY Configuration](https://res.cloudinary.com/hibbard/image/upload/w_auto,dpr_auto,q_auto/v1576058633/ubuntu-22.04-virtualbox-6.1.32/ubuntu-putty-1.png)

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
ssh-copy-id -i ~/.ssh/id_rsa.pub jim@192.168.178.49
```

Where `~/.ssh/id_rsa.pub` is the path to your public key, taken from the output above. And where `jim@192.168.178.49` should be altered to reflect your details.

The command will run and you should be asked for your server password. Enter it, then attempt to log into the server like so:

```sh
ssh jim@192.168.178.49
```

This time you should be in without a password.

## For Windows Users

You should be able to use a tool like [PuTTYgen](https://www.puttygen.com) to achieve the same thing. Here is a tutorial on using PuTTYgen to [create a new key pair for authentication](https://www.ssh.com/ssh/putty/windows/puttygen#sec-Creating-a-new-key-pair-for-authentication).

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
