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


