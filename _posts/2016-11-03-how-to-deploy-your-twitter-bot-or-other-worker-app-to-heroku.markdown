---
layout: post
title: "How to Deploy Your Twitter Bot (or Other Worker App) to Heroku"
date: 2016-11-03 07:00:00 -0500
image:
  feature: /assets/Screenshot-from-2016-10-30-14-12-08.png
categories:
- tutorials
tags:
- deployment
- heroku
- nodejs
- twitter bot
---
Ok, so we recently walked through getting started building a Node.js Twitter bot and then actually putting together functions to make it work. When we left off you had a really cool Twitter bot that acts automatically. Hopefully you've added some functions or features that really make it your own. Problem is, it's not much fun to have a Twitter bot that only works when your terminal is open.

<!--more-->

So now we're going to take that awesome Twitter bot you made and deploy it to a cloud platform called Heroku. This will enable your bot to always be working in the background, and we're going to do it for free.

At this point, you should have a Twitter bot that works when you run it locally. If you haven't already, go ahead and commit and push your most recent changes to your repository.

~~~ bash
$ git add .
$ git commit -m "some incredible commit message"
$ git push origin master
~~~

From here, we're going to go ahead and create a new local branch, so just on your machine, and give it some `<BRANCH-NAME>`. Branching essentially lets us work with one copy of your code without disturbing or changing the main copy. Git allows us to create and switch to the branch with one command in the terminal:

~~~ bash
$ git checkout -b <BRANCH-NAME>
~~~

Go ahead and open your `.gitignore` file and remove its reference to `config.js`.  While you don't want your authentication credentials being easily accessible on GitHub, you will need this file to deploy your app to Heroku.

Now, of course, if you don't already have a Heroku account, you can set one up for free [here](https://signup.heroku.com).

Once you're in and viewing your dashboard, create a new app, calling it whatever you like. Or leave it blank, and Heroku will come up with an interesting name for you.

Back to your terminal, you'll need to download the [Heroku Command Line Interface (CLI)](https://devcenter.heroku.com/articles/heroku-command-line) if you haven't already. *Note: You will need Ruby installed, as Heroku was originally designed for Ruby apps. There are a number of ways to do this, depending on what version you would like.*

Once it's installed, you will have access to your Heroku apps right from your terminal. Log into your account by entering the following into your terminal and following the prompts:

~~~ bash
$ heroku login
~~~

Now we're going to connect your working directory with your Heroku app:

~~~ bash
$ heroku git:remote -a <YOUR-APP-NAME>
~~~

And finally, we can deploy your application to Heroku. This should look very familiar to those who use Git and GitHub:

~~~ bash
$ git add .
$ git commit -am "add project files"
$ git push heroku <BRANCH-NAME>:master
~~~

Hmm, something about that was different, though. Remember that we want our master branch on Heroku to include the `config.js` file, but we *don't* want our master branch on GitHub to do so. So what we're doing in that last command is to tell Heroku that our local branch `<BRANCH-NAME> *is* the master branch.

Now technically, your Twitter bot has been deployed to Heroku. You'll notice, however, that if you look at your app dashboard online after a few minutes, it may say your app is sleeping. This is because Heroku assumes that if you build an app in Node.js, it will be a web app with a front-end for users to look at. It then uses that assumption to decide what kind dynos (Linux containers) it thinks you need.

This can be solved a few ways:

![Heroku web dyno off worker dyno on](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/Screenshot-from-2016-10-30-14-14-25.png)

Aside from adjusting it in the online GUI, one method is to create a `Procfile` (no extension, just that). Inside this file, we'll give instructions to Heroku about what this app should be doing:

~~~
worker: node bot.js
~~~

While I suggest including this file, when working with Heroku, it's a good idea to know how to change what dynos are being used on your app, so that you can correct or scale them if needed. This can be done right from your terminal:

~~~ bash
$ heroku ps:scale web=0 worker=1
~~~

After correcting the dynos or pushing changes, I usually restart the dyno I'm using. Heroku restarts your whole app after these changes, but sometimes working with the individual dyno keeps it from crashing on you:

~~~ bash
$ heroku restart worker.1
~~~

And from there, your app should work just like it did locally. If you need, you can use terminal commands to check on your app, like this one to check the status of your dynos:

~~~ bash
$ heroku ps
~~~

Or this one that shows your app console, any errors and shutdowns or restarts:

~~~ bash
$ heroku logs
~~~

Go ahead and check out your awesome app!

What kind of bot or app did you deploy? Feel free to share on [Twitter](https://twitter.com/intent/tweet?text=%40chznbaum&url=http%3A%2F%2Fmernmom.com%2F2016%2F11%2F03%2Fhow-to-deploy-your-twitter-bot-or-other-worker-app-to-heroku.html). Clicking that link will open a Twitter prompt directed at me including the link to this article for your convenience.