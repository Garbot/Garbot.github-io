#Building a Node.js Twitter API App

In my last post, I covered how to get an EC2 instance up and running with Node and Git.  Now we'll actually cover building the app in question: an app that displays random tweets from the patron saint of Twitter, [@dril](//www.twitter.com/dril).  

To get started, we'll download [this twitter library](//https://www.npmjs.com/package/twitter) for Node.  If you're building locally and then uploading to your EC2 instance, remember to install npm and this library there as well, or your code won't work.

The twitter API is a bit difficult to use.  There's essentially 3 steps we'll need to take here:

1. Create an application and register it with Twitter's API.
2. Obtain a bearer token.
3. 

### Registering your application

Go to (https://apps.twitter.com) and click "Create New App"

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

Fill out the form that appears.  If you don't have a website yet, no worries, just fill in a placeholder URL.  After creating your application, you'll just need to click on the "Keys and Access Tokens" tab to access your API Key.

Since we'll be using this a lot, you may want to declare your keys as variables in your Javascript file.  Store them in a **private** file, and reference them by requiring that file as a library.

I'm saving the following file as **keyinfo.js**.  It won't be included in my git repo:

```javascript
module.exports = {
  TWITTER_CONSUMER_KEY: "key goes here",
  TWITTER_CONSUMER_SECRET: "key goes here",
  ACCESS_TOKEN_KEY: "key goes here",
  ACCESS_TOKEN_SECRET: "key goes here",
  TWITTER_BEARER_TOKEN: ""
}
```

In your main application .js file, require this info by including the following line.

```javascript
var keyinfo = require('./keyinfo.js');
```

You can then easily access these keys by using object notation:

```javascript
keyinfo.TWITTER_CONSUMER_KEY
```

### Obtaining a bearer token

This is a bit obtuse.  You can read a detailed description of how to obtain a bearer token and why it's required [here](https://dev.twitter.com/oauth/application-only).  To make things easier, I'm just going to use this existing [script](https://gist.github.com/sulmanen/5245760) by [sulmanen](https://gist.github.com/sulmanen/5245760).  Input your key values into the appropriate variables - for example:

```javascript
var consumer_key = keyinfo.TWITTER_CONSUMER_KEY
var consumer_secret = keyinfo.TWITTER_CONSUMER_SECRET
```
Make sure you have the node request library installed.  If not...

``` npm install request ```

Once request is installed, run the script.  If you did everything properly, you should get a response back similar to this:

```javascript
{"token_type":"bearer","access_token":"BIG OL' KEY VALUE"}
```

Store this value in your **keyinfo.js** file as the object property TWITTER_BEARER_TOKEN or similar. 

### building the app

We're going to now build a very, very simple app.  First things first, let's grab the [twitter library for node](https://www.npmjs.com/package/twitter).

``` npm install twitter ```

Then 
