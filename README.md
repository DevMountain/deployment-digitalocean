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

##Pointing a Domain to your Droplet

First, you need to own the domain. Purchase it through GoDaddy, BlueHost, Namecheap, or whatever domain purchasing service you like best.

Once you own the domain, you need to edit the *nameservers* that domain is pointing to by default so they are pointing to your DigitalOcean droplet. Generally, this involves editing the nameservers so they look like this:

* *ns1.digitalocean.com*
* *ns2.digitalocean.com*
* *ns3.digitalocean.com*

Now, go into the DigitalOcean control panel and configure your domain. There are a few steps here, and they are probably best covered [here](https://www.digitalocean.com/community/articles/how-to-set-up-a-host-name-with-digitalocean).

A useful tool for checking your nameservers and DNS propagation: https://www.whatsmydns.net/

##Managing your Droplet and App

###Forever
Forever is a node module that ensures that a script continues to run, even if the SSH'd user logs out. To install forever, type:

```
sudo npm install -g forever
```

The commands you need to remember are these:

####`sudo -E forever start <script_name>`
This command starts a node script. The `-E` option will be important in a moment, because `-E` means that the root user (sudo) will pick up any environment variables that you have set. We'll set one in a future step.

####`sudo forever list`
Lists all currently running scripts. Important because you can see the log files associated with each script. Use `tail -f <path/to/log/file>` to watch the file.

####`sudo forever restartAll`
Restarts all currently running scripts. Important because you'll need to do this anytime the code changes (for example, from a `git pull`.

####`sudo forever stopall`
Stops all currently running scripts.

###Environment Variables
Often, you'll want to have settings that differ in testing from production. For example, which port should Express listen on? You'd want your app to listen on port 80 in production, but something like 8888 locally. We can accomplish this with environment variables.

To set an environment variable on the Linux OS on your droplet, edit the .bash_profile file in the user's home directory.

`nano ~/.bash_profile`

Edit the bash_profile file to set a variable. We'll create a variable called `EXPRESS_PORT`, but you could use a different name.

```
export EXPRESS_PORT=80
```

Now, to save this environment variable to your current environment, use `source ~/.bash_profile`

In your server.js file, you can use environment variables to specify a setting, such as a port:

```
app.listen(process.env.EXPRESS_PORT || 8888);
```

This way, EXPRESS_PORT will only be defined on our droplet, and will thus resolve to 80, while locally it will fall back to 8888.

You can use this same principle anywhere in your code where you have sensitive information you don't want saved in Github and that might differ from a local to a production environment. This could include database passwords, API keys/secrets, or ports.

*Additional resource by Chris Esplin: http://www.christopheresplin.com/single-page-app-architecture-angular-deploy*
