#Building a Node.js Twitter API App

In my last post, I covered how to get an EC2 instance up and running with Node and Git.  Now, let's build the app that will run on the EC2 server: an app that displays random tweets from the patron saint of Twitter, [@dril](//www.twitter.com/dril).  

To get started, download [this twitter library](//https://www.npmjs.com/package/twitter) for Node.  If you're building locally and then uploading to your EC2 instance, remember to install npm and this library there as well, or your code won't work.

I find Twitter's API to be somewhat difficult to use compared to other REST APIs.  You won't be able to make API calls using client side javascript.  Instead you'll need to use a server-side app as an intermediary for authentication purposes.  There's essentially 3 steps we'll need to take here:

1. Create an application and register it with Twitter's API.
2. Obtain an OAuth2 bearer token.
3. Create a server application to make API calls.
4. Create the front end of the app.

### Registering your application

Go to (https://apps.twitter.com) and click "Create New App"

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

Fill out the form that appears.  If you don't have a website yet, no worries, just fill in a placeholder URL.  After creating your application, you'll just need to click on the "Keys and Access Tokens" tab to access your API Key.

We'll be using these keys often, but we'll need to keep them private.  Don't include them anywhere in your JS file or anything that would be included in a public repo, such as on Github.  There are several ways to do this.  A good way of ensuring your program has access to these keys while protecting them from public access is to declare them as [process environment variables](https://www.linkedin.com/pulse/protect-your-api-keys-using-environment-variables-nodejs-dale-corns).

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

You can then easily access these keys as needed in your code by using object notation:

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

We're going to now build a simple app.  First things first, let's grab the [twitter library for node](https://www.npmjs.com/package/twitter).  Then include it in your app.

``` npm install twitter ```

``` var Twitter = require('twitter'); ```


#### Calling The API.
Twitter's search API doesn't index every tweet ever made by a user.  As such, it makes it difficult to grab a truly random tweet.  We'll approximate randomness by pulling a selection of recent tweets and pulling a random tweet from that group.  If anyone has found a good way to truly query the API on a random basis, please let me know!  Here is a script that will grab a number of tweets and return the id of a randomly selected tweet.  We'll then use this id in a callback function to get HTML that will display the tweet using an [embedded tweet](https://dev.twitter.com/web/embedded-tweets).

First, let's require the proper libraries and API keys.  If you set up your process environment variables correctly, they should be retrieved correctly by using the following code:

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
```

Next, let's establish the parameters for our request.  We want to retrieve tweets from @dril, and we'll test by retrieving 3 tweets to start with.  Later we'll up this parameter to 200.

```javascript
//establish parameters for our request
var params = {
  screen_name: 'dril',  //twitter handle
  count: 3              //number of tweets to return
};
```

Next, let's set up a web server using the http library.  You'll need to set the appropriate HTTP response headers.  To allow cross origin requests from a website, enter the URL.  You can alternatively enter `'*'` to allow access from any origin.

```javascript
//create web server.  Upon valid request, call the getTweet function.
var server = http.createServer(function(request, response) {
    response.setHeader('Content-Type', 'application/javascript');

    //see https://gist.github.com/balupton/3696140
    response.setHeader('Access-Control-Allow-Origin', 'http://drilquotes.s3-website.us-east-2.amazonaws.com/');
    response.setHeader('Access-Control-Request-Method', '*');
    response.setHeader('Access-Control-Allow-Methods', 'OPTIONS, GET');
    response.setHeader('Access-Control-Allow-Headers', '*');

    if ( request.method === 'OPTIONS' ) {
      response.writeHead(200);
      response.end();
      return;
    }
```


Net, let's build a function to query the [user timeline API](https://dev.twitter.com/rest/reference/get/statuses/user_timeline).  We'll pass in the params we declared earlier, along with a callback to execute upon receiving a response.  The response will contain an array of objects representing tweets from which we'll select the id of a random tweet.  We'll nest a call to a second callback within it, and call that callback with the selected tweet.

```
//function to call the user timeline API using our parameters.
//instead of returning the result, we give it to a callback function.
//the callback function will in this case write the tweet to the HTTP response.
function getTweet(callback){
  console.log("getting tweet");
  client.get('statuses/user_timeline', params, function(error, tweets, response) {
    if (!error) {
      var random = Math.floor(Math.random() * params.count);
      var selectedTweet = tweets[random].id_str;

      /*
       *Execute the callback function using the ID from the randomly selected tweet.
       *In this case, an anonymous function will be passed that will call a separate
       *Twitter API (Oembed) with the randomly chosen tweet ID.
       */
      callback(selectedTweet);

    } else {
      console.log("error:\n");
      console.log(error);
      callback(error);
    }
  });
}
```

If you want to see the full structure of a tweet object, feel free to log "tweets[number]" to the console.  So now that we have a random tweet ID, what next?  For the purpose of this app, I want to get the HTML to embed the tweet on my page.  Note the callback above.  we'll pass an anonymous function to the getTweet function that will execute upon completion and retrieve that HTML using the random tweet ID retrieved by getTweet().  This anonymous function will call a separate API, the Twitter [oembed API](https://dev.twitter.com/rest/reference/get/statuses/oembed).  This API does not require authentication, and will return embeddable HTML for our page.

We'll pass it a url based on the random ID we retrieved in the previous function.

```javascript
    /* pass an anonymous function as callback to getTweet.  in this callback, we write
     * the tweet data to the http response once the function has finished.  getTweet
     * will retrieve a random tweet id, which is passed to the anonymous function below.
     * The anonymous function then calls the oembed API using the parameters contained in
     * oembed_params and returns embeddable HTML.
     */
    getTweet(function(tweet){
      //variable data will be tweet id
      var oembed_params = {
        id: tweet,
        url: "https://twitter.com/dril/status/" + tweet
      }

      var finalTweet = "";

      client.get('https://publish.twitter.com/oembed', oembed_params, function(error, tweets) {
        if (!error) {
          finalTweet += tweets.html;

        } else {
          console.log("error:\n");
          console.log(error);
        }
        response.write(finalTweet);
        response.end();
      });



    });

});
```

Finally, let's instruct the server to listen on port 8080:

```
server.listen(8080);
```

Try hosting this locally, and then curling localhost:8080 from a new terminal window.  You should get a response with the tweet HTML!  When you eventually host the site, this will need to be changed to port 80 (HTTP) or 443 (HTTPS).

### Hosting the site

An easy way to port everything over to your EC2 instance is to just clone your git repo there.  Once you've done this, run the script we created earlier to initialize your process environment variables.  There's a couple things you need to keep in mind before running the app.  First, to enable access on any port below 1024, you'll need to run as root using sudo.  Since you're doing this, you'll also want to use the -E option to preserve environment variables.

```bash
sudo -E nodejs app.js
```

Note that this is not secure - if your site is compromised, the attacker wil have root access to your EC2 instance.  I'm not too worried about this in this instance.  If you are interested, however, [there are ways to mitigate this.](http://syskall.com/dont-run-node-dot-js-as-root/)  You should consider lowering privileges using process.setuid() once your app is running.

At this point, we're mostly finished.  You can find the full code of what we just built [here](https://github.com/Garbot/drilquotes/blob/master/app.js).

### Building the front end
Since we're already using AWS EC2 to host our app, let's use another AWS service to host the front end - S3 (Simple Storage Service).  S3 is great for hosting simple websites like the one we're building.  I won't go into the details of building the actual site in this post; I'll instead focus on how to get it up and running.

First, log into AWS and click on the link for S3.  You should be at the S3 Management Console.

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

The first step is to create a bucket.  Give it a name and select your preferred region for hosting.  Click on your newly created bucket and you'll have the option to upload.  In this case, I'll just upload the HTML and JS files for my site.

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

The next step is to make the files public.  Select them first, then select Actions->Make Public.

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

Once this is complete, go back to the console, select your bucket, then select properties at the top right.  Select the "Static Website Hosting" menu, and then enable website hosting.  Click on the provided URL, and you should now have access to your site!

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

### The final product
![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")
drilquotes.s3-website.us-east-2.amazonaws.com

We're finished!  That was a lot of work for a very simple app, but hopefully it will help illustrate to you the basics of running a Node server on AWS.
