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
