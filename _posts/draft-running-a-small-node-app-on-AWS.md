#Hosting a Small Node App on Amazon Web Services

Amazon Web Services is a great, free way to host a small app.

First things first, we'll need to spin up an EC2 instance.  For the purposes of this tutorial, we'll spin up an Ubuntu VM with a t2.micro CPU and 1 GB memory.  This is the default option presented, and allows for free usage.

Upon creating the instance for the first time, you'll be presented with a private key.  Save this somewhere where you can find it for future reference.  You'll need it to connect to the EC2 instance.  Click continue, and you should be directed to the EC2 Dashboard.  

![EC2 Dashboard](https://github.com/Garbot/Garbot.github.io/blob/master/img/aws-blog/1.png)

Once your instance is up and running, you'll need to do a few things to be able to connect.  First, navigate to the folder containing your key, and change its permissions - it can't be public.  The recommended permissions are as follows:

``` chmod 400 certificate.pem ```

Once this is done, you'll need to connect to your new server via SSH.

``` ssh -i "certificate.pem" ubuntu@INSERT-URL-HERE ```

You can obtain the URL by selecting your instance from the console, and then finding its Public DNS information under the "Description" tab.

![Public DNS information](https://github.com/Garbot/Garbot.github.io/blob/master/img/aws-blog/2.png)

Once you do this, you should find yourself logged into your server.

![EC2 command line](https://github.com/Garbot/Garbot.github.io/blob/master/img/aws-blog/3.png)

Good.  Let's go ahead and install node.

``` sudo apt-get install node ```

We're basically running a tiny Ubuntu VM, so you can use the regular *nix commands.  For example, let's create a directory.  It works as you expect it would.

``` mkdir test ```
``` ls ```

![EC2 command line 2](https://github.com/Garbot/Garbot.github.io/blob/master/img/aws-blog/1.png)

Instead of building my entire app from the command line on this tiny barebones VM, i'll just import an existing git repo and serve it from the EC2 instance.  First we'll need to install git.

``` sudo apt-get install git ```

Once the installation process is complete, we'll clone our repo.

``` git clone REPO-URL-HERE ```

You can then serve your app as you normally would.  I'll cover this in the next post. :)
