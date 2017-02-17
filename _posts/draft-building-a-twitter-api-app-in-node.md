#Building a Node.js Twitter API App

In my last post, I covered how to get an EC2 instance up and running with Node and Git.  Now we'll actually cover building the app in question: an app that displays random tweets from the patron saint of Twitter, [@dril](//www.twitter.com/dril).  

To get started, we'll download [this twitter library](//https://www.npmjs.com/package/twitter) for Node.  If you're building locally and then uploading to your EC2 instance, remember to install npm and this library there as well, or your code won't work.

The twitter API is somewhat difficult to use compared to other REST APIs.  You won't be able to make API calls using client side javascript.  There's essentially 3 steps we'll need to take here:

1. Create an application and register it with Twitter's API.
2. Obtain a bearer token.
3. Create a server application to make API calls.

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

This won't persist between sessions, however.  Once you close the terminal window, the variables you have set will disappear.  Instead, we can echo these keys to the local host profile in order to store them permanently.

```bash
$ echo set TWITTER_CONSUMER_KEY='your key here' >> ~/.profile
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
If you don't want to change the local host profile, so you can also write a script to do this on server startup if you so choose.


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
Twitter's search API doesn't index every tweet ever made by a user.  As such, it makes it difficult to grab a truly random tweet.  We'll approximate randomness by pulling a selection of recent tweets and pulling a random tweet from that group.  If anyone has found a good way to truly query the API on a random basis, please let me know!  We'll start with a barebones script that makes a request and logs the response to the console:

```javascript
var Twitter = require('twitter');

var client = new Twitter({
  consumer_key: process.env.TWITTER_CONSUMER_KEY,
  consumer_secret: process.env.TWITTER_CONSUMER_SECRET,
  access_token_key: process.env.ACCESS_TOKEN_KEY,
  access_token_secret: process.env.ACCESS_TOKEN_SECRET,
  bearer_token: process.env.TWITTER_BEARER_TOKEN
});

//establish parameters for our request
var params = {
  screen_name: 'dril',
  count: 200
};

//call the user timeline API using our parameters
client.get('statuses/user_timeline', params, function(error, tweets, response) {
  if (!error) {
    console.log(tweets);
  } else {
    console.log("error:\n");
    console.log(error);
  }
});
```

