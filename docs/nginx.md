In this part of tutorial we are going to securely run very simple website
using modern server `nginx`. You might know another popular webserver called
`apache`. Why have I chose `nginx` instead? It's mostly the matter of taste,
but my reasons are mainly these two:

* `nginx` is more recent (1995 vs 2004). Hence it could be build from the beginning as an asynchronous and non-blocking algorithm compared to `apache`.
* configuration in JSON like style instead of XML like

then there are some other technicalities (described e.g. on [DigitalOcean](https://www.digitalocean.com/community/tutorials/apache-vs-nginx-practical-considerations)) which are not going to really matter for what we want to achieve here. 

# Installing and setting nginx
As always we need to install appropriate package by `pacman -S nginx`. As you would probably already expect, a server needs to be running all the time and hence it's daemon. 
Start it by `systemctl start nginx` (and you might enable it too). If you now try to open `localhost` in your browser, you should be able to see nginx Welcome page.

## Forwarding ports
Let's first open `nginx` to world. For browsing websites on the internet the [HTTP(S)](https://www.instantssl.com/https-tutorials/what-is-https.html) protocols are used. These specify the default port, which is 80 for (insecure) HTTP and 443 for secure HTTPS. You need to forward this ports to the same ports on your server in the same way as you did in SSH chapter. 

If you were successful, you should be able to open see Welcome page when you enter your public IP address into your browser. 

## Configuration
The configuration file for `nginx` is stored in `/etc/nginx/nginx.conf`. Remember that lines starting with `#` are ignored, so relevant parts are those:

```
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
}
```
for now ignore everything except the `server` part. `listen` specify which port is going to be used for incoming connections and `server_name` is specifying to which domain/ipaddress this server corresponds. `location` defines the base directory with `index` which is called `index.htm(l)`. `/usr/share/nginx/html/index.html` is the Welcome page of `nginx`.

Now you should be able to understand why when you typed `localhost` to your browser you saw Welcome page. Your browser opened `localhost` on port 80 (default port - you can specify ports by typing them behind domain like `localhost:80`), `nginx` received this information and found appropriate server block. In this block you know that you should serve `/usr/share/nginx/html/index.html`. That's it!

### Creating a user who owns the website files
For sandboxing the content which should be served we will create a specific user. If an attacker gets a permissions for this user, he couldn't destroy the whole server, at least. Let's do it:
```
$ groupadd webdata
$ useradd -M -G webdata nginx -s /dev/null
```
No one will ever sign in as a `nginx` user, so we don't want to create a `home` directory for him (the `-M` switch) and he cannot run a shell (`-s /dev/null` switch assure this). Open `nginx` configuration file and insert `user nginx` on the very top of the file. Now all subprocesses will be spawned under this user, which again assures that they cannot access somewhere where nginx user cannot. 

We will now configure `nginx` to do something useful. Let's say we have this beautiful website which we want to serve using our new server:

```
<html>
<body>
<h1>My Welcome Page</h1>
</body>
</html>

```
Save this document in e.g. `/var/www/index.html` (you might need to create the `www` dir). Change the owner to the just created user by `chown nginx:webdata /var/www/index.html`. Let's edit an appropriate server blog so it will server our website now instead of the original welcome page. 

All you need to change is the `location` directive from `root /usr/share/nginx/html;` to `/var/www;`. 

To check if you wrote it correctly you can use `nginx`'s spell check using `nginx -t`. If everything is OK, run `nxinx -s reload`, which reloads the configuration.

### Creating HTTPS
The last part of this chapter will be a guide how to secure your connection with your own self-signed certificate. Even though our site doesn't need this, because there is no confidential information send out or in, it's very good to know how to handle this.

First, we need to create the certificate we are going to use. That can be done by two commands:
```
$ openssl genrsa -out privkey.pem 2048
$ openssl req -new -x509 -key privkey.pem -out cacert.pem -days 1095
$ mv *.pem /var/www # copy keys to www directory
$ chown nginx:webdata /var/www/*.pem  # change their owner to nginx user and webdata group
```

now we need to do small changes in our server block in `nginx.conf`. Add these lines right after the `listen 80;` directive:
```
listen 443 ssl;
ssl_certificate /var/www/cacert.pem;
ssl_certificate_key /var/www/privkey.pem;
```

and reload nginx by `nginx -s reload`. This simple change will make our connection through `HTTPS` secure while still leaving `http` connection possible. 

# Several other recommendations
These recommendations are so broad that that could have their own chapters as well, but it would be a bit outside of the scope of this guide. You should be already on the level where you could use other resources without problems, because we covered the basics. So, consider these:

1. update `nginx` frequently (this should be done with updating your system)
2. create all website in one directory (in their own sub directories), as we did in `/var/www` and assure that only `nginx` can access them. 
3. do not allow any other ports and servers in your configuration which are not necessary
