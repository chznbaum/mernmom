---
layout: post
title: "How to Build a Twitter Bot in Node.js - Part 2"
date: 2016-10-31 07:00:00 -0500
categories: tutorials, twitter bot
---
This is the second part of a two-part series on creating a Twitter bot. In the [first part]({{ site.baseurl }}{% post_url 2016-10-26-how-to-build-a-twitter-bot-in-nodejs-part-1 %}), we reviewed setting up Twitter credentials for the bot, ensured we have Node and NPM available, and began working with our directory structure and Twitter API module. In this second part, we'll go over using the API module to tweet, respond to tweets, follow and unfollow, and learn how to make the bot your own. Let's get started!

** Tweeting **

At this point, we are beginning to use the Twit module to create tweets. Reviewing the [Twit documentation](https://github.com/ttezel/twit), we can see that the first example uses a post method to create a new status - this is how we post a tweet.

Let's take a look at what this might look like as a reusable function within our code:

{% highlight javascript %}
var Twit = require('twit'); // Include Twit package
var config = require('./config'); // Include API keys
var T = new Twit(config);
var stream = T.stream('user'); // Set up user stream
function tweetIt(txt) { //Function to send tweets
  var tweet = { // Message of tweet to send out
    status: txt
  }
  T.post('statuses/update', tweet, tweeted); // Send the tweet, then run tweeted function
  function tweeted(err, data, response) { // Function to run after sending a tweet
    if (err) { // If error results
      console.log(err); // Print error to the console
    }
  }
}
tweetIt('Hello world!'); // Tweet "Hello world!"
{% endhighlight %}

This will allow us to pass as a parameter the content of your tweet to the function that will post the update. When the tweet attempt is made, we're going to check for errors, and print them to the console if they exist. In the case of our function call, the parameter we pass is `"Hello world!"`

You can open your terminal and run the bot by typing `nodejs bot.js` (or `node bot.js` for non-Linux systems). If you do, your bot should tweet "Hello world!" Go ahead and check it out.

This is a fairly basic tweet, as it only provides whatever text we select ahead of time, but with this structure, you can create functions to make the value you pass into `tweetIt()` more dynamic.

In order to be able to have any kind of interactive Twitter bot, though, we need to use some event listeners and respond accordingly. Let's check out our first one.

** Responding to Tweets **

The first event we're going to listen for is the "tweet" event, which will trigger a function to handle any processing that will take place. Our code will look like this:

{% highlight javascript %}
var Twit = require('twit'); // Include Twit package
var config = require('./config'); // Include API keys
var T = new Twit(config);
var stream = T.stream('user'); // Set up user stream
stream.on('tweet', tweetEvent); // Anytime a tweet enters the stream, trigger tweetEvent
function tweetEvent(eventMsg) { // Function to run on each tweet in the stream
  console.log(eventMsg);
}
{% endhighlight %}

Now, if you go into your terminal and restart the bot (type `.exit` to get out of the existing server before your `nodejs bot.js` command) and then tweet at your bot, your terminal console will fill with so much information that it can be hard to decipher. It contains things like details of who sent the message, any @mentions or geolocation information, information used to display a profile and other things.  This information is actually very useful in creating a bot that can respond to tweets, so let's try to make it look more legible:

{% highlight javascript %}
var Twit = require('twit'); // Include Twit package
var config = require('./config'); // Include API keys
var T = new Twit(config);
var stream = T.stream('user'); // Set up user stream
stream.on('tweet', tweetEvent); // Anytime a tweet enters the stream, trigger tweetEvent
function tweetEvent(eventMsg) { // Function to run on each tweet in the stream
  var fs = require('fs'); // Include the File System module
  var json = JSON.stringify(eventMsg, null, 4); // Prettify the eventMsg
  fs.writeFile('tweet.json', json); // Write the prettified eventMsg to a local file
}
{% endhighlight %}

Now if you save your file, run the bot again, and tweet at it, a file will be created called `tweet.json`. In it, you'll find what looks like a standard JSON file, and it becomes easier to tell what each piece of data actually is.

We're going to need the `user.screen_name`, `text`, and `in_reply_to_screen_name` properties in particular:

{% highlight javascript %}
var twit = require('twit'); // Include Twit package
var config = require('config'); // Include authentication credentials
var bot_screen_name = YOUR-BOT-NAME; // Set your bot's Twitter handle
var T = new Twit(config);
var stream = T.stream('user'); // Set up user stream
stream.on('tweet', tweetEvent); // Anytime a tweet enters the stream, trigger tweetEvent
function tweetEvent(eventMsg) { // Function to run on each tweet in the stream
  var from = eventMsg.user.screen_name; // Who sent the tweet
  var text = eventMsg.text; // Message of the tweet
  var reply_to = eventMsg.in_reply_to_screen_name; // Who tweet was @reply to
  if (from !== bot_screen_name) { // If bot didn't send the tweet
    text = text.replace(/[^a-zA-Z\s]/gi, '').toLowerCase(); // Remove non-letter characters and transform to lowercase
    var tweet_array = text.split(' '); // Create an array of each word in the tweet
    /*
    What to do to each tweet
    */
    if (reply_to !== null &amp;&amp; reply_to === bot_screen_name) { // If the tweet was @reply to bot
      /*
      What to do to each @reply
      */
    }
  }
}
{% endhighlight %}

Let's take a look at this for a moment. First is the addition of a new variable `bot_screen_name`. We're going to include your bot's handle in a few conditionals, so we may as well put it in a variable. In the body of the `tweetEvent()` function, we're going to set who sent the tweet, what it actually said, and if the tweet was an @reply, who it was in response to. Then we're going to check that the bot didn't send the tweet. This seems silly, but bear in mind that your bot's own tweets, follows and other events are a part of its stream. Inside that condition, we're also going to check to see if the tweet is an @reply, and if so if it is directed at your bot. With these conditions we can create logic to handle any tweets by those your bot follows as well as handling @replies.

Before I go into an example of how I handled mine, we need to add the tweeting function we used before. Remember that it looked a bit like this:

{% highlight javascript %}
function tweetIt(txt) { // Function to send tweets
  var tweet = { // Message of tweet to send out
    status: txt
  }
  T.post('status/update', tweet, tweeted); // Send the tweet, then run tweeted function
  function tweeted(err, data, response) { // Function to run after sending a tweet
    if (err) { // If error results
      console.log(err); // Print error to the console
    }
  }
}
{% endhighlight %}

Now to put it together, this is a simplified version of how I did my Mom Bot's tweet logic:

{% highlight javascript %}
var twit = require('twit'); // Include Twit package
var config = require('config'); // Include authentication credentials
var bot_screen_name = YOUR-BOT-NAME; // Set your bot's Twitter handle
var bad_words_list = require('badwords/array'); // Include Bad Words package
var unfollow_words_list = [
  'go away',
  /*
  ...
  */
  'leave me alone'
]
for (var k = 0; k &lt; bad_words_list.length; k++ {
  bad_words_list[k] = bad_words_list[k].toLowerCase();
}
var T = new Twit(config);
var stream = T.stream('user'); // Set up user stream
stream.on('tweet', tweetEvent); // Anytime a tweet enters the stream, trigger tweetEvent
function tweetEvent(eventMsg) { // Function to run on each tweet in the stream
  var from = eventMsg.user.screen_name; // Who sent the tweet
  var text = eventMsg.text; // Message of the tweet
  var reply_to = eventMsg.in_reply_to_screen_name; // Who tweet was @reply to
  if (from !== bot_screen_name) { // If bot didn't send the tweet
    text = text.replace(/[^a-zA-Z\s]/gi, '').toLowerCase(); // Remove non-letter characters and transform to lowercase
    var tweet_array = text.split(' '); // Create an array of each word in the tweet
    for (var i = 0; i &lt; tweet_array.length; i++) { // For each word in the tweet
      if (bad_words_list.indexOf(tweet_array[i]) != -1) { // If the word is in the bad words list
        tweetIt('@' + from + ' You tweeted a bad word! Mom Bot\'s not mad, she\'s just disappointed...'); // Bot tweets disappointment, example
      }
    }
    if (reply_to !== null &amp;&amp; reply_to === bot_screen_name) { // If the tweet was @reply to bot
      for (var j = 0; j &lt; unfollow_words_list.length; j++) { // For each word in the tweet
        if (text.indexOf(unfollow_words_list[j]) != -1) { // If an unfollow expression is in the tweet
          tweetIt('@' + from + ' Ok, I will leave you alone.'); // Tweet an unfollow response, example
        }
      }
    }
  }
}
function tweetIt(txt) { // Function to send tweets
  var tweet = { // Message of tweet to send out
    status: txt
  }
  T.post('status/update', tweet, tweeted); // Send the tweet, then run tweeted function
  function tweeted(err, data, response) { // Function to run after sending a tweet
    if (err) { // If error results
      console.log(err); // Print error to the console
    }
  }
}
{% endhighlight %}

Because of Mom Bot's "personality" she will scan all tweets issued by followers for "bad words" and issue a warning to the user if they occur.  For this, I used the Bad Words module to generate an array of words to check for.

![Mom Bot tweet response to bad words]{{{ site.url }}/assets/bad_word_screen.jpg)

In addition, she also scans @replies sent to her to see if they have any phrases set as unfollow-triggers. If she receives one, she will send a response that she will unfollow the user.

![Mom Bot Unfollow Tweet]({{ site.url }}/assets/unfollow_screen.jpg)

Of course, in my actual bot's code, I have a function randomizing the responses Mom Bot gives so that she's not constantly repeating herself. If you would like an example of the full code, you can always check out the source [here](https://github.com/chznbaum/the-mom-bot).

Note, she's not actually unfollowing them here. To do that, we're going to need to learn just a bit more.

** Follow and Unfollow on Command **

Now if we check the Twit documentation again, it has a couple of examples of using post methods to create and destroy friendships. Those are the methods we need, as those will enable the bot to follow and unfollow users.

In our case, we'll go ahead and follow each follower who follows the bot. It will look a bit like this:

{% highlight javascript %}
var twit = require('twit'); // Include Twit package
var config = require('config'); // Include authentication credentials
var bot_screen_name = YOUR-BOT-NAME; // Set your bot's Twitter handle
var bad_words_list = require('badwords/array'); // Include Bad Words package
var unfollow_words_list = [
  'go away',
  /*
  ...
  */
  'leave me alone'
]
for (var k = 0; k < bad_words_list.length; k++ {
  bad_words_list[k] = bad_words_list[k].toLowerCase();
}
var T = new Twit(config);
var stream = T.stream('user'); // Set up user stream
stream.on('tweet', tweetEvent); // Anytime a tweet enters the stream, trigger tweetEvent
stream.on('follow', followed); // Anytime a user follows bot, trigger followed
function tweetEvent(eventMsg) { // Function to run on each tweet in the stream
  var from = eventMsg.user.screen_name; // Who sent the tweet
  var text = eventMsg.text; // Message of the tweet
  var reply_to = eventMsg.in_reply_to_screen_name; // Who tweet was @reply to
  if (from !== bot_screen_name) { // If bot didn't send the tweet
    text = text.replace(/[^a-zA-Z\s]/gi, '').toLowerCase(); // Remove non-letter characters and transform to lowercase
    var tweet_array = text.split(' '); // Create an array of each word in the tweet
    for (var i = 0; i < tweet_array.length; i++) { // For each word in the tweet
      if (bad_words_list.indexOf(tweet_array[i]) != -1) { // If the word is in the bad words list
        tweetIt('@' + from + ' You tweeted a bad word! Mom Bot\'s not mad, she\'s just disappointed...'); // Bot tweets disappointment, example
      }
    }
    if (reply_to !== null &amp;&amp; reply_to === bot_screen_name) { // If the tweet was @reply to bot
      for (var j = 0; j < unfollow_words_list.length; j++) { // For each word in the tweet
        if (text.indexOf(unfollow_words_list[j]) != -1) { // If an unfollow expression is in the tweet
          tweetIt('@' + from + ' Ok, I will leave you alone.'); // Tweet an unfollow response, example
        }
      }
    }
  }
}
function followed(eventMsg) { // Function to run on follow event
  var follower_screen_name = eventMsg.source.screen_name; // Follower's screen name
  if (follower_screen_name !== bot_screen_name) { // If follower is not bot
    tweetIt('Thank you for following me!');
    T.post('friendships/create', { screen_name: follower_screen_name }, function(err, data, response) { // Follow the user back
      if (err) { // If error results
        console.log(err); // Print error to the console
      }
    });
  }
}
function tweetIt(txt) { // Function to send tweets
  var tweet = { // Message of tweet to send out
    status: txt
  }
  T.post('status/update', tweet, tweeted); // Send the tweet, then run tweeted function
  function tweeted(err, data, response) { // Function to run after sending a tweet
    if (err) { // If error results
      console.log(err); // Print error to the console
    }
  }
}
{% endhighlight %}

So here we've added an event listener to listen for follow events, which will trigger our `followed()` function. From there we check that the user doing the follow wasn't the bot, similar to how we checked tweets. If the event passes our check, the bot will follow the user and print an error if one is thrown.

![Mom Bot follow tweet]({{ site.url }}/assets/follow_screen.jpg)

Let's use what we've learned to actually unfollow the users we tweeted that we would earlier:

{% highlight javascript %}
var twit = require('twit'); // Include Twit package
var config = require('config'); // Include authentication credentials
var bot_screen_name = YOUR-BOT-NAME; // Set your bot's Twitter handle
var bad_words_list = require('badwords/array'); // Include Bad Words package
var unfollow_words_list = [
  'go away',
  /*
  ...
  */
  'leave me alone'
]
for (var k = 0; k < bad_words_list.length; k++ {
  bad_words_list[k] = bad_words_list[k].toLowerCase();
}
var T = new Twit(config);
var stream = T.stream('user'); // Set up user stream
stream.on('tweet', tweetEvent); // Anytime a tweet enters the stream, trigger tweetEvent
stream.on('follow', followed); // Anytime a user follows bot, trigger followed
function tweetEvent(eventMsg) { // Function to run on each tweet in the stream
  var from = eventMsg.user.screen_name; // Who sent the tweet
  var text = eventMsg.text; // Message of the tweet
  var reply_to = eventMsg.in_reply_to_screen_name; // Who tweet was @reply to
  if (from !== bot_screen_name) { // If bot didn't send the tweet
    text = text.replace(/[^a-zA-Z\s]/gi, '').toLowerCase(); // Remove non-letter characters and transform to lowercase
    var tweet_array = text.split(' '); // Create an array of each word in the tweet
    for (var i = 0; i < tweet_array.length; i++) { // For each word in the tweet
      if (bad_words_list.indexOf(tweet_array[i]) != -1) { // If the word is in the bad words list
        tweetIt('@' + from + ' You tweeted a bad word! Mom Bot\'s not mad, she\'s just disappointed...'); // Bot tweets disappointment, example
      }
    }
    if (reply_to !== null &amp;&amp; reply_to === bot_screen_name) { // If the tweet was @reply to bot
      for (var j = 0; j < unfollow_words_list.length; j++) { // For each word in the tweet
        if (text.indexOf(unfollow_words_list[j]) != -1) { // If an unfollow expression is in the tweet
          tweetIt('@' + from + ' Ok, I will leave you alone.'); // Tweet an unfollow response, example
          T.post('friendships/destroy', { screen_name: from }, function(err, data, response) { // Unfollow the user
            if (err) { // If error results
              console.log(err); // Print error to the console
            }
          });
        }
      }
    }
  }
}
function followed(eventMsg) { // Function to run on follow event
  var follower_screen_name = eventMsg.source.screen_name; // Follower's screen name
  if (follower_screen_name !== bot_screen_name) { // If follower is not bot
    tweetIt('Thank you for following me!');
    T.post('friendships/create', { screen_name: follower_screen_name }, function(err, data, response) { // Follow the user back
      if (err) { // If error results
        console.log(err); // Print error to the console
      }
    });
  }
}
function tweetIt(txt) { // Function to send tweets
  var tweet = { // Message of tweet to send out
    status: txt
  }
  T.post('status/update', tweet, tweeted); // Send the tweet, then run tweeted function
  function tweeted(err, data, response) { // Function to run after sending a tweet
    if (err) { // If error results
      console.log(err); // Print error to the console
    }
  }
}
{% endhighlight %}

Now at least the bot is being honest!

** Make it Your Own **

Now the bot can follow and unfollow independently, it can respond to tweets, and you can create its own tweets fairly easily. But it's not terribly creative at this point since it basically follows what my bot does, and even then, it doesn't do quite as much.

The full example of my bot is below, and as you can see, it includes additional responses and randomizes them. It will respond to @replies with certain "sad" or "proud" words. It also includes some instances where it will notify me if the bot isn't able to do what's expected.

{% highlight javascript %}
console.log('The bot is starting...');
var Twit = require('twit'); // Include Twit Package
var config = require('./config'); // Include authentication credentials
var bot_name = 'Mom Bot';
var bot_screen_name = 'the_mother_bot';
var bot_owner_name = 'otherconsolelog';
var disappointed_bot_list = [ // What Bot will say when user tweets a bad word
  bot_name + "'s not mad; she's just disappointed. 😞 Would you like to delete that tweet?",
  "You're an adult, but do you really want to keep that tweet? 🤔 You can delete it if you don't want a future boss to see.",
  "Do you kiss your Mother Bot with that mouth? 😠 You can always delete that tweet if you think it was foolish.",
  "Sweetie, your tweet had a bad word! 😲 Are you sure that's what you want people to see? You can delete it if not."
];
var thank_you_list = [ // What Bot will say when user follows Bot
  "Thank you for keeping in touch with " + bot_name + ", Sweetie. 😘",
  "Don't tell your brother, but of course you're my favorite, Sweetie. 😍"
];
var unfollow_bot_list = [ // What Bot will say when Bot unfollows user
  "Ok Sweetie, " + bot_name + " will give you space. Follow me again if you want to talk. 😥",
  bot_name + " always loves you, but I\'ll leave you alone. Follow me again if you want to talk. 😥",
  "Ok Sweetie, " + bot_name + " will miss you, but I'm glad you're having a good time. 😥"
];
var feel_better_bot_list = [ // What Bot will say when user tweets about sadness
  "I'm happy simply because you exist. Just thought you should know that. 😘",
  "You are the light of my world and the first thing I think about every day. 😘",
  "Always remember that you are needed and there is work to be done. 😉",
  "Keep in mind that this, too, will pass. In the meantime, I am here for you in whatever way you need. 🤗"
];
var proud_bot_list = [ // What Bot will say when user tweets about something great/to be proud of
  "I'm so proud of you. And even if you weren't so fantastic, I'd still be proud. 😆",
  "I believe in you, Sweetie. 😘",
  "You are one of the best gifts I've ever gotten. I am so proud and humbled. 😊",
  "I feel so proud when I'm with you. 😊",
  "You have some real gifts! 😆",
  "It is so cool to watch you grow up. 😆",
  "You make me so happy just by being you. 😊",
  "I love you so much!😘",
  "You were born to do this! 😆"
];
var unfollow_words_list = [ // Words user can include to request Bot to unfollow the user
  "all your fault",
  "dont care",
  "dont have to",
  "dont need you",
  "go away",
  "hate you",
  "leave me alone",
  "not my mom",
  "not my real mom",
  "run away"
];
var sad_words_list = [ // Words user can include to request cheering up
  "blue",
  "blah",
  "crestfallen",
  "dejected",
  "depressed",
  "desolate",
  "despair",
  "despairing",
  "disconsolate",
  "dismal",
  "doleful",
  "down",
  "forlorn",
  "gloomy",
  "glum",
  "heartbroken",
  "inconsolable",
  "lonely",
  "melancholy",
  "miserable",
  "mournful",
  "sad",
  "sorrow",
  "sorrowful",
  "unhappy",
  "woebegone",
  "wretched"
];
var proud_words_list = [ // Words user can include to express happiness/pride
  "accomplished",
  "accomplishment",
  "amazing",
  "awesome",
  "cheering",
  "content",
  "delighted",
  "glad",
  "glorious",
  "good",
  "grand",
  "gratified",
  "gratifying",
  "happy",
  "heartwarming",
  "inspiring",
  "joyful",
  "magnificent",
  "memorable",
  "notable",
  "overjoyed",
  "pleased",
  "pleasing",
  "proud",
  "resplendent",
  "satisfied",
  "satisfying",
  "splendid",
  "succeeded",
  "success",
  "thrilled"
];
var bad_words_list = require('badwords/array'); // Include Bad Words package
for (k = 0; k < bad_words_list.length; k++) {
  bad_words_list[k] = bad_words_list[k].toLowerCase(); // Transform Bad Words list to all lowercase
}
var T = new Twit(config);
var stream = T.stream('user'); // Setting up a user stream
stream.on('tweet', tweetEvent); // Anytime a tweet enters the stream, run tweetEvent
stream.on('follow', followed); // Anytime a user follows Bot, run followed
console.log('Entering the stream.');
function tweetEvent(eventMsg) { // Function to run on each tweet in the stream
  var from = eventMsg.user.screen_name; // Who sent the tweet
  var text = eventMsg.text; // Message of the tweet
  var reply_to = eventMsg.in_reply_to_screen_name; // Who tweet was @reply to
  if (from !== bot_screen_name) { // If Bot didn't send the tweet
    console.log('Bot received a tweet.');
    text = text.replace(/[^a-zA-Z\s]/gi, "").toLowerCase(); // Remove non-letter characters and transform to lowercase
    var tweet_array = text.split(' '); // Create an array of each word in the tweet
    for (var i = 0; i < tweet_array.length; i++) { // For each word in the tweet
      if (bad_words_list.indexOf(tweet_array[i]) != -1) { // If the word is included in bad words list
        var disappointed_text = randomSaying(disappointed_bot_list);
        console.log('That tweet had a bad word!');
        tweetIt('@' + from + ' ' + disappointed_text); // Bot tweets her disappointment
      }
    }
    if (reply_to !== null &amp;&amp; reply_to === bot_screen_name) { // If the tweet was @reply to Bot
      for (var j = 0; j < unfollow_words_list.length; j++) { // For each word in the unfollow list
        if (text.indexOf(unfollow_words_list[j]) != -1) { // If an unfollow word is in the tweet
          var unfollow_text = randomSaying(unfollow_bot_list);
          console.log('Someone wanted to unfollow.');
          tweetIt('@' + from + ' ' + unfollow_text); // Tweet an unfollow response
          T.post('friendships/destroy', { screen_name: from }, function(err, data, response) { // Unfollow the user
            if (err) { // If error results
              console.log(err); // Print error to the console
              tweetIt('@' + from + ' Something\'s wrong with ' + bot_name + '\'s computer. Ask @' + bot_owner_name + ' to help me unfollow you, please.'); // Tweet a request for user to contact Bot Owner
            }
          });
        }
      }
      for (var l = 0; l < tweet_array.length; l++) { // For each word in the tweet
        if ('stop'.indexOf(tweet_array[l]) != -1) { // If 'stop' is in the tweet
          console.log('Someone\'s having a problem.');
          tweetIt('@' + from + ' ' + bot_name + ' seems to be upsetting you. Please ask @' + bot_owner_name + ' for help.'); // Tweet a request for user to contact Bot Owner
        } else if (sad_words_list.indexOf(tweet_array[l]) != -1) { // If a sad word is in the tweet
          var feel_better_text = randomSaying(feel_better_bot_list);
          console.log('Someone needs cheering up.');
          tweetIt('@' + from + ' ' + feel_better_text); // Tweet to cheer the user up
        } else if (proud_words_list.indexOf(tweet_array[l]) != -1) { // If a proud word is in the tweet
          var proud_text = randomSaying(proud_bot_list);
          console.log('Someone did something awesome.');
          tweetIt('@' + from + ' ' + proud_text); // Tweet to be proud of the user
        }
      }
    }
  }
}
function followed(eventMsg) { // Function to run on follow event
  console.log('Someone followed the bot.');
  var name = eventMsg.source.name; // Who followed
  var follower_screen_name = eventMsg.source.screen_name; // Follower's screen name
  if (follower_screen_name !== bot_screen_name) { // If follower is not Bot
    var thank_you = randomSaying(thank_you_list)
    tweetIt('@' + follower_screen_name + ' ' + thank_you); // Tweet a thank you expression
    T.post('friendships/create', { screen_name: follower_screen_name }, function(err, data, response) { // Follow the user back
      if (err) { // If error results
        console.log(err); // Print error to the console
      }
    });
  }
}
function tweetIt(txt) { // Function to send tweets
  var tweet = { // Message of tweet to send out
    status: txt
  }
  T.post('statuses/update', tweet, tweeted); // Send the tweet, then run tweeted function
  function tweeted(err, data, response) { // Function to run after sending a tweet.
    if (err) { // If error results
      console.log(err); // Print error to the console
    }
  }
}
function randomSaying(sayingList) { // Function to randomize the expression to use
  var saying_number = Math.floor(Math.random()*sayingList.length); // Give a random number within number of available responses
  var saying = sayingList[saying_number]; // Grab expression matching that number
  return saying; // Return the expression
}
{% endhighlight %}

![Mom Bot cheer up tweet]({{ site.url }}/assets/cheer_up_screen.jpg)

![Mom Bot proud tweet]({{ site.url }}/assets/proud_screen.jpg)

Go ahead and take some time to think about what kinds of things you would like your bot to do and how you might implement that logic.

Enjoy learning things visually? [Daniel Shiffman](http://shiffman.net/), creator of Coding Rainbow has a fantastic [YouTube video series](https://www.youtube.com/playlist?list=PLRqwX-V7Uu6atTSxoRiVnSuOn6JHnq2yV) you should check out that helped me a great deal with Twitter bots. If you feel extra awesome, you can help support his work on [Patreon](https://www.patreon.com/codingrainbow) or buy one of his books, [The Nature of Code: Simulating Natural Systems with Processing](https://www.amazon.com/gp/product/0985930802) or [Learning Processing, Second Edition: A Beginner's Guide to Programming Images, Animation, and Interaction](https://www.amazon.com/gp/product/0123944430).

Let me know what kind of bot you've made, share it so I can follow it, or ask questions in the comments!