---
layout: default
title: "Hello World!"
type: post
navigation: false

date: 2016-07-26 14:49:00 -0500
excerpt: These two words have been typed countless times as developers take their first steps in new languages. So they should be fitting now, as I begin to chronicle my own path from fledgling coder to full-stack developer. This is the start of what is proving to be a fascinating, if vexing, journey. To begin, I should probably take a moment to share how this blog came to be, particularly since it almost didn’t happen.
categories:
- code log
gradient: 1
image: nightcomputer.jpg
details: true

author: Chazona Baum
bio: When you're a mom in a male-dominated industry, it can sometimes be hard to have someone in your corner or who understands your challenges. I had my kids before learning to code, and I started my tech career while they were still pre-school age. I've literally led code meetups with my toddler around my ankles. While our experience may not be identical, I've got your back on this JavaScript journey.
twitter: "https://twitter.com/chznbaum"
linkedin: "https://linkedin.com/in/chznbaum"

published: true
---
These two words have been typed countless times as developers take their first steps in new languages. So they should be fitting now, as I begin to chronicle my own path from fledgling coder to full-stack developer. This is the start of what is proving to be a fascinating, if vexing, journey.

To begin, I should probably take a moment to share how this blog came to be, particularly since it almost didn’t happen. Given the value and necessity of GitHub, I chose to host my page using their [GitHub Pages](https://pages.github.com/) tool. Of course, as it seems everyone new to code finds out on discovering GitHub’s utility, there is a steep learning curve to the tools. I ultimately created and deleted the site twice, staying up until 2 AM to set it up, so I hope that my experience makes it easier for someone else.

## Forget Spooning; It’s Time to Fork ##

{% include
  media-image.html
  file="hubpress.PNG"
  title="Get App: Fork the hubpress.io repository and get started!"
  caption="Hubpress setup" %}

For my setup, I chose to use Hubpress to manage my blog. Other options are available, but they typically require pushing content using the command line, while Hubpress uses a dashboard interface. If you choose to go this same route, the most imporant thing is to fork (copy) the Hubpress repository to your GitHub account.

**Do not** create your GitHub page first. Once the repository is forked, you can rename it in the style `.github.io` that GitHub Pages would have used. You need to retain the fork relationship with Hubpress.io to ensure you can apply updates later on, which won’t happen if you push everything to an empty repository.

{% include
  media-image.html
  file="rename.PNG"
  title="GitHub Pages Settings Page - Options"
  caption="Rename your page here, if this is a user page" %}

## What’s In a Name — a Headache ##

If you intend to use a custom domain, this is a good time to ensure it is ready to use. I used [Google Domains](https://domains.google/) for my blog. Google Domains is in Beta right now, but my experience with it has been smooth so far. Most domains are available for $12 a year with free private listing and easy integration of services like [Apps for Work](https://apps.google.com/).

{% include
  media-image.html
  file="configjson.PNG"
  title="Hubpress config.json file meta"
  caption="Edit your config.json file" %}

From there, you can go into the Code section of your repository, under `/hubpress/config.json`. You should fill in as follows and merge the changes in your local master branch:

~~~
"username": ,
"repositoryName": ,
"branch": "master",
~~~

{% include
  media-image.html
  file="pagesdomain.PNG"
  title="GitHub Pages Settings: Custom Domain"
  caption="Enter your custom domain" %}

Go back into settings and scroll down to enter your custom domain in the applicable field. Click save.

Back in the root of your repository, enter the file `CNAME` and make sure your domain is listed on two lines, as follows:

~~~
domain.com
www.domain.com
~~~

Once this is all complete on the GitHub side, you can sync up your domain. If you used Google Domains, this part is manageable.

{% include
  media-image.html
  file="customresourcerecords.PNG"
  title="Google Domains Custom Resource Records"
  caption="Enter your GitHub info" %}

While managing `My domains`, click `DNS` and scroll all the way to the bottom. For your site to work with or without the `www` subdomain, you should include a `CNAME` entry with the `www` subdomain and your `.github.io` URL.

Do not use a * or *wildcard* subdomain to set up your GitHub blog. This will allow **anyone** to create their own page under your primary domain.

You also need two A records pointing to GitHub’s IPs. At this time, they are `192.30.252.153` and `192.30.252.154`.

These IPs can change over time. Confirm the current ones at [GitHub Apex Domain Help](https://help.github.com/articles/setting-up-an-apex-domain/) before pointing your domain to them.

## Sweet, Sweet, Sweet Victory ##

From there, it may take a few moments or up to 24 hours for everything to point where it should. When you view your site, if you see the following screen, you have set everything up correctly.

{% include
  media-image.html
  file="successfulsetup.PNG"
  title="Hubpress.io: An awesome blog will ne here soon!"
  caption="Hooray for confirmation!" %}

Add `/hubpress` to the end of your domain to access the dashboard login screen, and use your GitHub username and password. From there, you will see a blank page — your posts screen — and a menu bar you can use to access settings like your blog’s title and social media links.

You must set your blog’s title before you start trying to save posts.

Congratulate yourself on a job well done, as this process trips many people up. Once you’re ready to write, pull up your favorite editor like Notepad++, brush up on [AsciiDoc's plain text syntax](http://asciidoctor.org/docs/asciidoc-writers-guide/) and you’re on your way.

How was your first experience jumping into GitHub? Share your experiences in the comments!