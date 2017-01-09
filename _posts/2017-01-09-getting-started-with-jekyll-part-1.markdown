---
layout: post
title: "Getting Started with Jekyll - Part 1"
date: 2017-01-09 07:00:00 -0500
image: https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/jekyll-logo-light-solid.png
categories:
- tutorials
tags:
- blogging
- capistrano
- jekyll
- nginx
- ruby
published: true
---
If you're a developer, there's a good chance you may have heard of static site generators like [Jekyll](https://jekyllrb.com/) or [Ghost](https://ghost.org/). They claim to cut down on a lot of the bloat you find in behemoths like WordPress and simultaneously give you greater control over your site's structure and design.

For those very reasons, I have recently migrated my existing WordPress blog over to Jekyll. While WordPress is a massively useful tool for sites that require a lot of features, it's simply unnecessary for a simple blog.

For those who have never managed their blog in this way, though, it can be pretty intimidating. Thankfully, there are a couple of ways to go about installing Jekyll, and both are manageable.

The easiest way to install Jekyll is to use [GitHub Pages](https://pages.github.com). I won't go over that method here; instead, to have more control over the installation and site itself, we're going to walk through hosting Jekyll on your own *server*, an always-connected computer for serving content, or *Virtual Private Server*/*VPS*, a piece of a server blocked off from the rest that acts like its own server. If you have never managed your own server before, this is a great project to learn from!

By the end of Part 1, you're going to have a functioning server you can add a domain name to. In Part 2, we'll get to the dependencies needed to make Jekyll and our deployments work. Finally in Part 3, we will configure Jekyll and get our site ready to deploy some posts. Ready for some fun?

## Requirements ##

To follow along with this tutorial, you will need a server or VPS with *root* access. Basically this means you need administrator access to view and change things like system files on your server. Not sure who to use? My recommendation is [Digital Ocean](https://digitalocean.com); for $5 a month, you will have more server than you need, full root access, as well as great documentation and support. This tutorial assumes you are able to use a command-line terminal, text editor like Sublime Text or Atom, and a web browser. It also assumes Linux, though folks using other systems are welcome to go through and let me know where you get stuck so we can work through it. You do *not* need to already know any particular language (though comfort with Ruby and Bash help), so if anything doesn't make sense, please bring it to my attention.

## Set Up Your VPS ##

To start with, you're going to need to do some basic setup of your server. In this tutorial, we're using Ubuntu 16.04, which Digital Ocean and others VPS providers make easy to install while creating your server. Once you receive your root login information (likely by email), log into your VPS using SSH on your console. Grab your server's *IP address*, a series of four or six numbers separated by periods that tells your browser where to find your server. For Digital Ocean, this can be found on the `Droplets` page like this:

![Droplets: Name, IP Address, Created, Tags](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/DropletsScreenshot.png)

You should type something like the following, filling in your server's IP address in place of the zeros:

    $ ssh root@00.00.00.00

You'll likely see something like the following:

    The authenticity of host '00.00.00.00 (00.00.00.00)' can't be established.
    ECDSA key fingerprint is a0:b1:c2:d3:e4:f5:g6:h7:i8:j9:k0:l1:m2:n3:o4:p5.
    Are you sure you want to continue connecting (yes/no)?

Type `yes` and enter your root password:

    $ root_password

Your terminal should require you to immediately change your password. Enter your original password and then the new one twice. The prompt should then change to reflect your using the droplet like so: `root@00.00.00.00 ~$`.

Now, this root user can easily get you in a lot of trouble because it has access to *everything* on your server and can do *anything* to it. You don't want to be the person who accidentally deletes everything on your server by hitting just a few keystrokes.

So what we're going to do instead is create a new user that can access administrator privileges, but only deliberately so accidents are less likely to be catastrophic. Choose a username, and in your terminal, type:

    $ adduser user_name

You will be prompted for a password, and then you'll see additional information requested:

    Changing the user information for user_name
    Enter the new value, or press ENTER for the default
      Full Name []:
      Room Number []:
      Work Phone []:
      Home Phone []:
      Other []:
    Is the information correct? [Y/n]

Enter what you like; it's okay to leave them blank if you prefer. Then we can get to adding privileges. In your terminal, type:

    $ usermod -aG sudo user_name

This will add `user_name` to the user group `sudo` which by default has *superuser* or administrator privileges. To access those privileges, when you're issuing a command you'll type `sudo` in front of it, like this:

    $ sudo command

The first time you use this in each session, you will have to enter your user's account password. The prompt will look like:

    [sudo] password for user_name:

Before we log out of the root user and into the one you've created, it's a good idea to connect an *SSH key pair* to the server. These are a set of codes in which one is stored locally (the *private* key) and one is stored on the server (the *public* key). In order to log in with them, the user must have the right private key that matches up with the server's public key. These codes are much harder to crack than passwords, so they make your server more secure against brute-force attacks.

If you don't already have a key pair, you'll need to go ahead and create one. In a separate local terminal window (not connected to your server), type the following:

    $ ssh-keygen -t rsa

You will see a couple of prompts:

    Enter file in which to save the key (/home/user_name/.ssh/id_rsa):
    Enter passphrase (empty for no passphrase):

For the first prompt, just hit `enter`. The second is largely up to you. A passphrase could potentially provide more security, say if your local computer was hacked or stolen, but you would need to enter it every time you use the key pair.

When the keys have been created, the public key will be stored at `/home/user_name/.ssh/id_rsa.pub` and the private key will be at `/home/user_name/.ssh/id_rsa`. These can be stored in a password manager vault as a backup. Now that you have them, you'll need to copy the public key to your server under each of your accounts. While still in your local terminal window, type the following, replacing `00.00.00.00` with your server's IP address:

    $ ssh-copy-id root@00.00.00.00

You should see something like this:

    The authenticity of host '00.00.00.00 (00.00.00.00)' can't be established.
    RSA key fingerprint is a0:b1:c2:d3:e4:f5:g6:h7:i8:j9:k0:l1:m2:n3:o4:p5.
    Are you sure you want to continue connecting (yes/no)?

Type `yes`. Then you'll see:

    Warning: Permanently added '00.00.00.00' (RSA) to the list of known hosts.
    root@00.00.00.00's password:

Enter the password for @user_name. You'll see:

    Now try logging into the machine, with "ssh 'root@00.00.00.00'", and check in:

      ~/.ssh/authorized_keys

    to make sure we haven't added extra keys that you weren't expecting.

At this point, in your server terminal window, you can go ahead and log out of the root account and try to log back in:

    $ exit
    $ ssh root@00.00.00.00

You should not be prompted for a password, though you will for a passphrase if you created one. Go ahead and do these same steps again starting with copying the public key to your non-root user, so that both accounts can be accessed without a password.

*Important: Only complete this next step if you were **successfully** able to log in without the password on your accounts. Otherwise you can lock yourself out of your own server.*

The next thing you can do, which is optional but generally a good idea, is to disable password-logins for your root account. This ensures that someone won't be able to gain administrator access to your server just because they cracked your password. If you do this step, using your server terminal window, open the config file for SSH:

    $ sudo nano /etc/ssh/sshd_config

Look for `PermitRootLogin` and modify it so it says:

    PermitRootLogin without-password

Then to make the changes effective, type this into your terminal to reload SSH:

    $ sudo service ssh reload

Now that we have this out of the way, there isn't much need for you to log in to the root account, so get out of there and log into your non-root account.

## Configure Your Server with Nginx ##

At this point, you should be logged into your server as your non-root user. Now, we're going to install the Nginx web server, which will be pretty easy. In your terminal, type:

    $ sudo apt-get update
    $ sudo apt-get install nginx

`apt-get` is a package manager that will help us install things; as long as the name of the program we need is in its listings, it should be able to install it for us. By updating it first, we're making sure we have the most up-to-date listings to pull from. The second command handles the actual installation, including any needed dependencies. *Note: If you get a dependency error installing anything in this tutorial, you may need to look at the error and install the dependency prior to installing the program you want. Just be prepared.*

Now's a pretty good time to make sure we have a *firewall* set up, since we'll need to make sure Nginx can get through it. A firewall keeps track of traffic in and out of your server and makes sure that only traffic that meets certain rules can get through. The tool we're going to use to manage our firewall is `ufw`, which comes built-in.

First, go ahead and check the status of your firewall by typing:

    $ sudo ufw status verbose

You should get a message stating it is inactive. We're going to deliberately enable it. First, we need to make sure that traffic can get in from the *ports* we need. Ports are specific endpoints for traffic in and out of the server; for example, emails are generally handled from a different port than your published content will be.

First, since we are accessing our server remotely through *SSH*, a secure shell within our terminal, we need to make sure we can keep accessing that way:

    $ sudo ufw allow ssh

Since SSH requires port 22, that command will allow connections from that port. We'll also make sure that http connections (the kind you usually make in your browser) are allowed. Like with SSH, `ufw` knows what port to listen to for http connections (80), so you can type:

    $ sudo ufw allow 'Nginx HTTP'

If you have or get a SSL security certificate for your site, which will connect users by https instead and is generally a best practice these days, you will also need to keep its port open:

    $ sudo ufw allow 'Nginx HTTPS'

We're not going to worry about *File Tranfer Protocol*/*FTP* connections, used for transferring files back and forth with your server with no encryption (not a great idea anyway), or other ports because we're going to deploy our site's files right from SSH.

Now that we have our needed ports open, go ahead and enable the firewall:

    $ sudo ufw enable

If you go back and check the firewall status again, you should get output like this:

    Status: active

    To                         Action      From
    --                         ------      ----
    OpenSSH                    ALLOW       Anywhere                  
    Nginx HTTP                 ALLOW       Anywhere 

Now we should be able to check with our system to see that Nginx is running. Type:

    $ systemctl status nginx

You'll get a detailed output, but the piece you want to look for is `Active: active (running) since`. Just to be sure, go ahead and see if you get Nginx's default landing page. In your browser window's navigation bar, type your server's IP address (should look like `00.00.00.00`) and hit `enter`. You should see something that looks like this:

![Welcome to nginx!](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/nginx_landing_page.png)

If you did, great! Your server is working correctly.

We're finished for today, Thursday we'll get all the dependencies needed and install Jekyll, and Monday we'll get the blog deployed! Want something to do in the meantime? If you have Digital Ocean, here's a useful [guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean) by [Etel Sverdlov](https://twitter.com/etelsverdlov) on setting up your domain/hostname on your server.

Questions, comments or concerns? Continue the conversation on [Twitter](https://twitter.com/intent/tweet?text=%40chznbaum&url=http%3A%2F%2Fmernmom.com%2F2017%2F01%2F09%2Fgetting-started-with-jekyll-part-1.html). Clicking that link will open a Twitter prompt directed at me including the link to this article for your convenience.