Deploying a Node.js app to DigitalOcean
=======================

These steps assume that you've already created a DigitalOcean account.

##How to add a SSH key
See if you already have an ssh public key generated:

```
cd ~/.ssh
ls
```

If you see either id_dsa.pub or id_rsa.pub, then you're good. If not, you need to generate one.

```
ssh-keygen -t rsa -C "myemail@example.com"
```

You can decide whether or not you want to add a password to this ssh key.

Once the key is generated, "cat" it by doing:

```
cat ~/.ssh/id_rsa.pub
```
(yours might be `id_dsa.pub`)

Copy the text spit out by the cat command and paste it into the large textarea that shows up in DigitalOcean when you click SSH Keys > Add SSH Key. Give it a name, something like "My Macbook Pro."

##How to SSH into your machine
* Make sure you've completed the payment form and created a Droplet (you can use all the defaults) and that you have your *root password* that you should have received via email.
* use the following command to SSH into the machine instance: `ssh root@<ip address>`
For example:

```
ssh root@107.251.123.654
```
You will be prompted for the root password.

##How to add a user to your Ubuntu machine

It is not generally advisable to access machines habitually as the user "root." The best practice generally is to utilize another user with less privileges, and then just use `sudo` when you need to.

Create a user by using this command: `useradd -m -s /bin/bash -U <username>`

Here's an example:

```
useradd -m -s /bin/bash -U homersimpson
```

###Optional: remove password restriction for new user

If you're like me, you hate throwing passwords around. I prefer to use my SSH key only and not have to rely on stored passwords. (In an enterprise environment, this would be different).

To make it so you can ssh into your machine as the new user you created without a password, do the following:
* Make sure you're SSH'd into your machine as root.
* Create a .ssh directory for your user:

```
mkdir /home/<your_username>/.ssh
```

* Copy your SSH key from your machine (remember `cat`) and use this command while SSH'd into your droplet:

```
echo "BIG_LONG_SSH_KEY" >> /home/<your_username>/.ssh/authorized_keys
```

Now you can SSH into your droplet as the new user you created: `ssh <your_username>@<ip address>`

* You can also set it up so this user doesn't need a password when using `sudo`. Again, as root on your droplet:

```
sudo visudo
```

This will open a text editor on the command line. This file specifies which groups have sudo access. See the admin group? We're going to make your user a part of that group in a second, but first let's allow the admin group to use sudo without a password.

* Edit the line with the %admin group so it reads like this:

```%admin ALL=(ALL) NOPASSWD: ALL```

Save the file by hitting ctrl-x, the Y to save.

* Now create the admin group and add the user to that admin group (again, as root on the droplet):

```
addgroup admin
adduser <your_username> admin
```

Now you can SSH into your droplet with your username/SSH public key and have sudo powers.

##How to install Node.js on your Droplet
* SSH into your droplet with a user that has sudo privileges (see above)
* The Ubuntu package manager, "apt-get" is a very handy tool. It will install almost anything we need. But with Node.js, we need a few extra steps:

```
sudo apt-get update
sudo apt-get install -y python-software-properties python g++ make
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```

The whole reason we're doing the above commands is because by default, apt-get installs a pretty old version of Node.js.

Here's what we just did. The first line (update) updates the package manager. Always the first thing you do on an Ubuntu box. The second line installs some basic software we'll need to continue. The third line (add-apt-repository) adds a new source list that includes the latest Node packages. Then we update apt-get again, and finally install nodejs.

To double check everything worked, check which version you have installed by typing `node --version`. It should be greater than 0.10. (It also includes npm, which doesn't happen when you install using the default source list for apt-get).

##Installing Git on your Droplet
* OK, this is tough ... get ready ...

```
sudo apt-get install git
```

That's it. Expecting more? apt-get is pretty smooth.

##Running your app from git on your droplet

Here's where it gets tricky. Or at least, there are a lot of possible ways to do what's coming up. I'll show you an easy way, and more involved way. But first, let's check out your project via git so it's really easy to update.

* SSH into your droplet
* clone your git repository for your app into a directory (could be /home/<your username>/app or /home/<your username>/code or whatever you'd like)

Now, when you make updates while you're developing locally, you'll push to github. Then to update the production server, you'll SSH into that machine and then `pull` from that server to get the changes.

###Deploying your app the "easy" way
Basically, we need to get your app running and listening on port 80 for web requests from your user's browsers. How you get your app listening on port 80 varies greatly depending on what you're using (express, sails, grunt, etc.). 

For SailsJS you need to edit the config/local.js file to specify a different port (80) than the default. You can read more about running SailsJS in production [here](http://sailsjs.org/#!documentation/deployment). 

For a basic express app, if you're specifying the port somewhere in your app.js or server.js, just change it to listen on port 80 rather than whatever port you usually listen on in development.

For grunt/yeoman, you're going to need to dive into the Gruntfile.js to determine how best to run and deploy your app from port 80. This might be challenging, so you could move onto the next section ...

###Deploying your app the "less easy" way
We can use nginx to do all of the hardcore server stuff for us. Nginx is a server software that's tested and true, and handles a lot of things very well, including things like SSL (https). We're going to use Nginx as a "reverse proxy" for our app, meaning Nginx will listen for port 80 for us, then forward any relevant requests to our app.

####Installing/Configuring Nginx

```
sudo apt-get install nginx
```

hooray!

Let's start Nginx and behold it in all its glory:

```
sudo service nginx start
```

Now navigate to your droplet's IP address in your browser and you should see a message that says "Welcome to nginx!"

Now we need to create a file that represents the "site" we're serving. Remember, on a single Ubuntu/server instance or droplet we could be serving multiple sites at once from different directories. This is one reason that using Nginx to route web requests will be very helpful. (In the previous step you could only have one app listening on port 80, so you couldn't deploy another.)

Let's create the file:

```
sudo nano /etc/nginx/sites-available/<your sitename>
```

Generally, I replace sitename with the domain name of the site I'm serving. But it can be whatever you'd like. `nano` will open up a text editor for this file. Here's a good structure for this file to start with:

```
server {
  listen 80;

  root /home/<your_username>/path/to/code;
  index index.html index.htm;

  server_name example.com;
}
```

This essentially creates a static server for serving files from a directory. If you have no API or server-side code that needs to run, you could be finished right here. However, if you are running an API/express/Sails/mean.io app, you'll need to configure this site to pass on requests to Node:

```
server {
  listen 80;
  
  location / {
    proxy_pass         http://127.0.0.1:1337/;
    proxy_redirect     off;

    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
  }
```

See how we're using `proxy_pass` to send all requests to a localhost address? This means that all we need to do is run our app pretty much like we would on our local machine, and Nginx will send it requests and return its responses to the user.

Make sure you have the port (above is 1337) configured correctly.

Once you have the above file configured, we need to copy the file into the /sites-enabled directory, which is the directory nginx actually uses to look for hosts. Kind of silly it has to be in two places, I know, but we can make that easier by creating a symlink (a shortcut) from this file we created to the one that needs to be in /sites-enabled, that way if we edit one, we are actually editing it in both places.

```
sudo ln -s /etc/nginx/sites-available/<your sitename> /etc/nginx/sites-enabled/<your sitename>
```

Now, make sure your app is running. I recommend using the "forever" service to run it. (Forever is an npm module that keeps a node file running, even if it crashes for some reason):

```
sudo npm install -g forever
forever start <your-main-script-file>.js
```

Now restart Nginx:

```
sudo service nginx restart
```

And if everything went well you'll see your app running the next time you hit that IP address from the browser!

*Additional resource by Chris Esplin: http://www.christopheresplin.com/single-page-app-architecture-angular-deploy*

##Pointing a Domain to your Droplet

First, you need to own the domain. Purchase it through GoDaddy, BlueHost, Namecheap, or whatever domain purchasing service you like best.

Once you own the domain, you need to edit the *nameservers* that domain is pointing to by default so they are pointing to your DigitalOcean droplet. Generally, this involves editing the nameservers so they look like this:

* *ns1.digitalocean.com*
* *ns2.digitalocean.com*
* *ns3.digitalocean.com*

Now, go into the DigitalOcean control panel and configure your domain. There are a few steps here, and they are probably best covered [here](https://www.digitalocean.com/community/articles/how-to-set-up-a-host-name-with-digitalocean).

A useful tool for checking your nameservers and DNS propagation: https://www.whatsmydns.net/
