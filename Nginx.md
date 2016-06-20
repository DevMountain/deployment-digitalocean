# Dealing with nginx and subdomains

## Setting up the subdomain

First thing you need to do is go into the digital ocean web interface and go to Networking > Domains.  

From there you can click the magnifying glass next to your domain (If you have no domains follow the steps in the main README.md in this repository.

Now we need to add an 'A' record.  Enter the subdomain you want to use and the IP address of your server.

## Nginx & Ubuntu

Follow the steps here if you don't already have nginx :

[Setup Guide](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-14-04-lts)

## Modifying the nginx default file

Navigate to /etc/nginx/sites-available

There should be a file present called default

Open the file in edit mode (You may need to `sudo nano -f default` to get permissions)

Copy one of the server blocks and modify it to match.  The basic structure looks like this.

```
server {
  listen 80;  //This is where requests come in, on port 80.  Leave this alone
  server_name surveys.devmounta.in surveys.devmountain.com; //Update this to use your subdomain
  location / {
    proxy_pass         http://127.0.0.1:8002/; //Make sure this matches the port you're running on on this server
    proxy_redirect     off;  //The rest of these are default and can be left alone
    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
  }
}
```


The only other gotcha is if you're using sockets.  Then you need to use a location structure that looks like this :
```
location / {
    proxy_pass         http://127.0.0.1:8003/;
    proxy_redirect     off;
    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_http_version 1.1;
    proxy_set_header   Upgrade $http_upgrade;
    proxy_set_header   Connection "upgrade";
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
}
```

Save your changes!

## Restarting nginx

We need to bounce the nginx service so it picks up our changes :

`sudo service nginx restart`

This will wipe all sessions and force everyone using this server to have to re-log in.  So keep this command to a minimum please.
