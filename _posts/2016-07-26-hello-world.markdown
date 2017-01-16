---
layout: post
title: "Hello World!"
date: 2016-07-26 14:49:00 -0500
image:
  feature: /assets/nightcomputer.jpg
categories:
- code log
---
These two words have been typed countless times as developers take their first steps in new languages. So they should be fitting now, as I begin to chronicle my own path from fledgling coder to full-stack developer. This is the start of what is proving to be a fascinating, if vexing, journey.

To begin, I should probably take a moment to share how this blog came to be, particularly since it almost didn’t happen.<!--more--> Given the value and necessity of GitHub, I chose to host my page using their [GitHub Pages](https://pages.github.com/) tool. Of course, as it seems everyone new to code finds out on discovering GitHub’s utility, there is a steep learning curve to the tools. I ultimately created and deleted the site twice, staying up until 2 AM to set it up, so I hope that my experience makes it easier for someone else.

## Forget Spooning; It’s Time to Fork ##

![Get App: Fork the hubpress.io repository and get started!](http://chazonabaum.com/images/hubpress.PNG)
Figure 1. Hubpress setup

For my setup, I chose to use Hubpress to manage my blog. Other options are available, but they typically require pushing content using the command line, while Hubpress uses a dashboard interface. If you choose to go this same route, the most imporant thing is to fork (copy) the Hubpress repository to your GitHub account.

**Do not** create your GitHub page first. Once the repository is forked, you can rename it in the style `.github.io` that GitHub Pages would have used. You need to retain the fork relationship with Hubpress.io to ensure you can apply updates later on, which won’t happen if you push everything to an empty repository.

![GitHub Pages Settings Page - Options](http://chazonabaum.com/images/rename.PNG)  
Figure 2. Rename your page here, if this is a user page

## What’s In a Name — a Headache ##

If you intend to use a custom domain, this is a good time to ensure it is ready to use. I used [Google Domains](https://domains.google/) for my blog. Google Domains is in Beta right now, but my experience with it has been smooth so far. Most domains are available for $12 a year with free private listing and easy integration of services like [Apps for Work](https://apps.google.com/).

![Hubpress config.json file meta](http://chazonabaum.com/images/configjson.PNG)  
Figure 3. Edit your config.json file

From there, you can go into the Code section of your repository, under `/hubpress/config.json`. You should fill in as follows and merge the changes in your local master branch:

~~~
"username": ,
"repositoryName": ,
"branch": "master",
~~~

![GitHub Pages Settings: Custom Domain](http://chazonabaum.com/images/pagesdomain.PNG)  
Figure 4. Enter your custom domain

Go back into settings and scroll down to enter your custom domain in the applicable field. Click save.

Back in the root of your repository, enter the file `CNAME` and make sure your domain is listed on two lines, as follows:

~~~
domain.com
www.domain.com
~~~

Once this is all complete on the GitHub side, you can sync up your domain. If you used Google Domains, this part is manageable.

![Google Domains Custom Resource Records](http://chazonabaum.com/images/customresourcerecords.PNG)  
Figure 5. Enter your GitHub info

While managing `My domains`, click `DNS` and scroll all the way to the bottom. For your site to work with or without the `www` subdomain, you should include a `CNAME` entry with the `www` subdomain and your `.github.io` URL.

Do not use a * or *wildcard* subdomain to set up your GitHub blog. This will allow **anyone** to create their own page under your primary domain.

You also need two A records pointing to GitHub’s IPs. At this time, they are `192.30.252.153` and `192.30.252.154`.

These IPs can change over time. Confirm the current ones at [GitHub Apex Domain Help](https://help.github.com/articles/setting-up-an-apex-domain/) before pointing your domain to them.

## Sweet, Sweet, Sweet Victory ##

From there, it may take a few moments or up to 24 hours for everything to point where it should. When you view your site, if you see the following screen, you have set everything up correctly.

![HubPress.io: An awesome blog will be here soon!](http://chazonabaum.com/images/successfulsetup.PNG)  
Figure 6. Hooray for confirmation!

Add `/hubpress` to the end of your domain to access the dashboard login screen, and use your GitHub username and password. From there, you will see a blank page — your posts screen — and a menu bar you can use to access settings like your blog’s title and social media links.

You must set your blog’s title before you start trying to save posts.

Congratulate yourself on a job well done, as this process trips many people up. Once you’re ready to write, pull up your favorite editor like Notepad++, brush up on [AsciiDoc's plain text syntax](http://asciidoctor.org/docs/asciidoc-writers-guide/) and you’re on your way.

How was your first experience jumping into GitHub? Share your experiences on [Twitter](https://twitter.com/intent/tweet?text=%40chznbaum&url=http%3A%2F%2Fmernmom.com%2F2016%2F07%2F26%2Fhello-world.html). Clicking that link will open a Twitter prompt directed at me including the link to this article for your convenience.