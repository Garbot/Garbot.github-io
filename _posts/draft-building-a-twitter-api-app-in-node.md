#Building a Node.js Twitter API App

In my last post, I covered how to get an EC2 instance up and running with Node and Git.  Now we'll actually cover building the app in question: an app that displays random tweets from the patron saint of Twitter, [@dril](//www.twitter.com/dril).  

To get started, we'll download [this twitter library](//https://www.npmjs.com/package/twitter) for Node.  If you're building locally and then uploading to your EC2 instance, remember to install npm and this library there as well, or your code won't work.

The twitter API is somewhat difficult to use compared to other REST APIs.  You won't be able to make API calls using client side javascript.  Instead you'll need to use a server-side app as an intermediary for authentication purposes.  There's essentially 3 steps we'll need to take here:

1. Create an application and register it with Twitter's API.
2. Obtain an OAuth2 bearer token.
3. Create a server application to make API calls.
4. Create the front end of the app.

### Registering your application

Go to (https://apps.twitter.com) and click "Create New App"

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

Fill out the form that appears.  If you don't have a website yet, no worries, just fill in a placeholder URL.  After creating your application, you'll just need to click on the "Keys and Access Tokens" tab to access your API Key.

We'll be using these keys often, but need to keep them private.  Don't include them anywhere in your JS file or anything that would be included in a public repo, such as on Github.  There are several ways to do this.  A good way of ensuring your program has access to these keys while protecting them from public access is to declare them as [process environment variables](https://www.linkedin.com/pulse/protect-your-api-keys-using-environment-variables-nodejs-dale-corns).

For example:

```bash
$ export set TWITTER_CONSUMER_KEY='your key here'
$ echo $TWITTER_CONSUMER_KEY
>your key here
```

This won't persist between sessions, however.  Once you close the terminal window, the variables you have set will disappear.  I'll use a script to make this process easier on startup:

```bash
#!/bin/bash
export set TWITTER_CONSUMER_KEY="KEY";
export set TWITTER_CONSUMER_SECRET="KEY";
export set ACCESS_TOKEN_KEY="KEY";
export set ACCESS_TOKEN_SECRET="KEY";
export set TWITTER_BEARER_TOKEN="KEY";
```

Before running your node app, run the script.  Make sure to include the [source command](http://unix.stackexchange.com/questions/30189/how-can-i-make-environment-variables-exported-in-a-shell-script-stick-around):

```bash
source script.sh
```

In your main application .js file, you can now access this information like so:

```javascript
var client = new Twitter({
  consumer_key: process.env.TWITTER_CONSUMER_KEY,
  consumer_secret: process.env.TWITTER_CONSUMER_SECRET,
  access_token_key: process.env.ACCESS_TOKEN_KEY,
  access_token_secret: process.env.ACCESS_TOKEN_SECRET,
  bearer_token: process.env.TWITTER_BEARER_TOKEN
});
```

You can then easily access these keys as needed by using object notation:

```javascript
client.consumer_key;
```


### Obtaining a bearer token

This is a bit obtuse.  You can read a detailed description of how to obtain a bearer token and why it's required [here](https://dev.twitter.com/oauth/application-only).  To make things easier, I'm just going to use this existing [script](https://gist.github.com/sulmanen/5245760) by [sulmanen](https://gist.github.com/sulmanen/5245760).  Input your key values into the appropriate variables - for example:

```javascript
var consumer_key = process.env.TWITTER_CONSUMER_KEY
var consumer_secret = process.env.TWITTER_CONSUMER_SECRET
```
Make sure you have the node request library installed.  If not...

``` npm install request ```

Once request is installed, run the script.  If you did everything properly, you should get a response back similar to this:

```javascript
{"token_type":"bearer","access_token":"BIG OL' KEY VALUE"}
```

Save this value somewhere private, and initialize it as a process environment variable like we did earlier.

### building the app

We're going to now build a very, very simple app.  First things first, let's grab the [twitter library for node](https://www.npmjs.com/package/twitter).  Then include it in your app.

``` npm install twitter ```

``` var Twitter = require('twitter'); ```


### Calling The API.
Twitter's search API doesn't index every tweet ever made by a user.  As such, it makes it difficult to grab a truly random tweet.  We'll approximate randomness by pulling a selection of recent tweets and pulling a random tweet from that group.  If anyone has found a good way to truly query the API on a random basis, please let me know!  Here is a pretty simple script that will grab a number of tweets and return the id of a randomly selected tweet.  We'll then use this id in the client-side javascript to display the tweet using an [embedded tweet](https://dev.twitter.com/web/embedded-tweets).

```javascript
var Twitter = require('twitter');
var http = require('http');

var client = new Twitter({
  consumer_key: process.env.TWITTER_CONSUMER_KEY,
  consumer_secret: process.env.TWITTER_CONSUMER_SECRET,
  access_token_key: process.env.ACCESS_TOKEN_KEY,
  access_token_secret: process.env.ACCESS_TOKEN_SECRET,
  bearer_token: process.env.TWITTER_BEARER_TOKEN
});

//establish parameters for our request
var params = {
  screen_name: 'dril',  //twitter handle
  count: 200            //number of tweets to return
};


//function to call the user timeline API using our parameters.
//instead of returning the result, we give it to a callback function.
//the callback function will in this case write the tweet to the HTTP response.
function getTweet(callback){
  console.log("getting tweet");
  client.get('statuses/user_timeline', params, function(error, tweets, response) {
    if (!error) {
      var random = Math.floor(Math.random() * params.count);
      var selectedTweet = tweets[random].id_str;
      callback(selectedTweet);
    } else {
      console.log("error:\n");
      console.log(error);
      callback(error);
    }
  });
}

//create web server.  Upon valid request, call the getTweet function.
var server = http.createServer(function(request, response) {
    response.setHeader('Content-Type', 'text/plain');
    //pass an anonymous function as callback to getTweet.  in this callback, we write
    //the tweet data to the http response once the function has finished.
    getTweet(function(data){
      response.write(data);
      response.end();
    });
});

server.listen(8080);
```

Try hosting this locally, and then curling localhost:8080 from a new terminal window.  You should get a response with the tweet ID!  When you eventually host the site, this will need to be changed to port 80 (HTTP) or 443 (HTTPS)

### Hosting the site

An easy way to port everything over to your EC2 instance is to just clone your git repo there.  Once you've done this, run the script we created earlier to initialize your process environment variables.  There's a couple things you need to keep in mind before running the app.  First, to enable access on any port below 1024, you'll need to run as root using sudo.  Since you're doing this, you'll also want to use the -E option to preserve environment variables.

```bash
sudo -E nodejs app.js
```

Note that this is not secure - if your site is compromised, the attacker wil have root access to your EC2 instance.  [There are ways to mitigate this, however.](http://syskall.com/dont-run-node-dot-js-as-root/)  You should consider lowering privileges using process.setuid() once your app is running.

### Building the front end
Since we're already using AWS EC2 to host our app, let's use another AWS service to host the front end - S3 (Simple Storage Service).  S3 is great for hosting simple websites like the one we're building.
