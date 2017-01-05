---
layout: post
title: "How to Build a Twitter Bot in Node.js - Part 1"
date: 2016-10-26 07:00:00 -0500
categories: tutorials, twitter bot
---
When just starting out with Node.js, piecing together a front- and back-end and successfully deploying the app can be a bit intimidating. A great way to get your feet wet working with Node.js and deploying to Heroku without having a front-end to deal with is by making a Twitter bot. Not only are they incredibly fun, but because they're only logic, they make a great stepping stone to bigger projects.

In this two part series, I'll walk you through creating an interactive Twitter bot step by step, with examples from my own bot. My example bot, called [Mom Bot](https://twitter.com/the_mother_bot), follows and unfollows users independently and scans tweets and @replies so it can respond accordingly. The source code and documentation for my bot can be viewed in its entirety [here](https://github.com/chznbaum/the-mom-bot).

For the first part, we'll review setting up Twitter credentials for the bot, ensure we have Node and NPM available, and begin working with our directory structure and Twitter API module. In the second part, we'll go over using the API module to tweet, respond to tweets, follow and unfollow, and learn how to make the bot your own.

## Getting Set Up on Twitter ##

First things first: please don't use your primary Twitter account for this. There are exceptions, say if you are making a bot to handle follows or @replies for you, but for the most part, you don't want to risk your main account being flagged as spam.

Once you've created an account for your bot, head over to the [Twitter Developers](https://dev.twitter.com) page. This is the place for documentation on making all kinds of Twitter apps. In the navigation bar, navigate to "My Apps."

![Twitter Developers Navigation and Header](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/Screenshot-from-2016-10-27-00-38-04.png)

Log in as your bot if you aren't already, and click to "Create an Application." You can name your application anything you like, but try to keep your name and description relevant to your bot. Your website can be anything; my suggestion is your repository URL for the bot on GitHub. Read and accept the Twitter Developer Agreement and click to "Create your Twitter Application."

![Create an Application Form](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/create_an_application.jpg)

From there, you'll see several tabs of information and settings. The only one that matters to us for this bot is the "Keys and Access Tokens" tab.

![Keys and Access Tokens Tab](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/keys_and_access_tab.jpg)

Go ahead and generate your Access Token and Access Token Secret, and then leave this window open; we'll need the four codes on this page in just a few minutes.

## Node.js and NPM ##

![Node JS](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/Node.js_logo.png)

### Linux ###

For those of you running a Linux box, the great news is that your distro may come with Node.js and the Node Package Manager (NPM) already installed. A good way to check if you already have it is to open your terminal. Go ahead and enter:

    $ nodejs

If it comes preinstalled, your terminal should become a console prompt, shown by the `>`  symbol. You can work with JavaScript in this console just like you would in your browser's console. Go ahead and check it out! When you're ready to exit the console, simply type `.exit` or hit the `Ctrl` and `C` buttons twice.

You could also just as easily check for a man page about nodejs:

    $ man nodejs

Similarly, to check if NPM is installed, you can check for its man page:

    $ man npm

If you already have both of these tools installed, go ahead to the next section. Let the others catch up.

### Mac ###

Mac users, while you have some extra installations ahead of you, they're not all that difficult,

First, make sure you have XCode installed. This is a free download from the Apple App Store, and it will allow you to compile software for your Mac.

The next Mac-specific install will be Homebrew, which is another package manager. To install it, go to your terminal, type the following, and follow the prompts:

    $ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

Once you have Homebrew, installing Node.js and the NPM are fairly simple. In your terminal, simply type the following:

    $ brew install node

Homebrew should have installed both Node.js and NPM. To verify this, enter two commands just checking the version of each piece of software:

    $ node -v
    $ npm -v

If you received version numbers, congrats! Move ahead to the next section and let those running Windows get to here.

### Windows ###

Windows users, we're actually only going to use the shell or command line to test whether your installation was successful. Go ahead to the [Node.js web site](https://nodejs.org/en/download/), download and run their Windows installer (either [32-bit](https://nodejs.org/dist/v4.6.1/node-v4.6.1-x86.msi) or [64-bit](https://nodejs.org/dist/v4.6.1/node-v4.6.1-x64.msi) ).

You should be finished at this point. Go ahead and check for version numbers by entering the following in your terminal:

    &gt; node -v
    &gt; npm -v

If you got version numbers, you're ready to move on.

## Starting the Actual Project ##

Now we can actually start building the app. As with almost any Node.js project, we want to start by creating our `package.json` file. NPM actually makes that easy for us. Go ahead and enter the following into your terminal, and follow the prompts to create this file.

    $ npm init

This file now holds some key information about our app, and it will eventually include any dependencies as well.

The next file we'll create is our `bot.js` file. This will take the place of the `server.js` file in most Node.js projects and will be the source of all the logic for our bot.

Before we start writing the logic, though, we're going to need to include a library of functions for interacting with tweets and streams. In this case, we're going to use Twit, an API module that is useful for bot-making. To install it in your project, go back to your terminal and enter:

    $ npm install twit --save

That `--save` term will include the module in the dependencies of your `package.json` file.

![NPM Twit Module](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/Screenshot-from-2016-10-27-01-00-43.png)

Of course, in order for your bot to do anything with a Twitter account, it needs to authenticate. You'll need to create a `config.js` file to hold your API keys because you *don't* want them to be in your main logic file. Remember the Twitter screen with the four codes? Enter them in the `config.js` file like this:

{% highlight javascript %}
module.exports = {
  consumer_key: '...',
  consumer_secret: '...',
  access_token: '...',
  access_token_secret: '...'
}
{% endhighlight %}

Now, let's start building the bot. In your `bot.js` file, first you need to actually include the Twit module you just installed and the `config.js` file you just created:

{% highlight javascript %}
var Twit = require('twit'); // Include Twit package
var config = require('./config'); // Include API keys
var T = new Twit(config);
var stream = T.stream('user'); // Set up user stream
{% endhighlight %}

The last two lines get you set up to start listening to your account's Twitter stream for events like tweets and follows.

When you're ready to start piecing this bot together, go ahead to Part 2. In the meantime, feel free to get familiar with the Twitter API module we're using, [Twit](https://github.com/ttezel/twit).