<h1 align="center">
  Secure VPS
</h1>

<h4 align="center">Creating a secure VPS</h4>

## Contents

* [Deciding for a VPS hosting service](#deciding-for-a-vps-hosting-service)
* [Why use nginx?](#why-use-nginx?)
* [Why use LetsEncrypt?](#why-use-letsEncrypt?)
* [Create a Pre-Installed VPS](#create-a-pre-installed-vps)
* [Set up Initial Security Level](#set-up-initial-security-level)
* [Initial Nginx Configuration](#initial-nginx-configuration)

### Deciding for a VPS hosting service

  
### Why use nginx?
* Nginx currently has the best way to manage resources.
* Nginx creates a new thread of process on every request. 
* Every request is not dependent from other requests.
* Allowing for faster and more secure requests.
* Normal version of apache put all requests in the same tree, 
meaning that if some reason, a request takes more time, 
the other requests are going to fail.
* Making child requests makes that request tree vulnerable for DDOS attacks.
* Nginx don't use unsecure files like .htaccess etc.


### Why use LetsEncrypt?
* Free: Anyone who owns a domain name can use Let’s Encrypt to get a trusted certificate at zero, zilch, nada cost.
* Automatic: Software running on a webserver can interact with Let’s Encrypt to painlessly obtain a certificate, securely configure it for use, and automatically take care of renewal.
* Secure: Let’s Encrypt will advance TLS security best practices, both on the Certificate Authority (CA) side and by helping site operators properly secure their servers.
* Transparent: All certificates issued or revoked will be publicly recorded and available for anyone to inspect.
* Open: The automatic issuance and renewal protocol will be published as an open standard that others can adopt.
* Cooperative: Much like the Internet protocols themselves, Let’s Encrypt is a joint effort to benefit the community, beyond any one organization’s control.


### VPS
* Create a Pre-Installed VPS on your chosen hosting service. i.e AWS, Digital Ocean etc.
* SSH on from your local computer to your VPS, set up the needed permission and rights on your VPS.
* Buy a domain.
* Configure the new domain (and subdomains) inside your hosting provider's network and put the ip of your VPS in there.
* Install all needed dependencies on your VPS, preferably use some automation tools like ansible or terraform.
* LEMP stack, git, php modules, composer etc.


### Set up Initial Security Level
* Generate SSH public and private keys for secure connection between your local machine.
`ssh-keygen -t rsa -b 4096 -C "root"`
* Never use the root account for normal everyday work! Make users that are delegated on every task.
`adduser <username>`
* Start a session of the newly creates user.
* Give admin permission to the newly creates user.
`sudo adduser <username> sudo`
* Start a session of the new user that's now on the sudo group.
* Add permissions the right way.
`sudo mkdir permissions/test`
`stat permissions`
* Provide the default user permission.
`sudo chown -R www-data permissions/`
* Check permission then mak a folder without sudo:
`stat permissions`
`mkdir permissions/test2`
* Always specify a specific permission for a specific user!
* Enable firewall.
`sudo apt-get install ufw`
`sudo ufw status`
`sudo ufw default deny incoming`
`sudo ufw default allow outcoming`
`sudo ufw allow ssh`
* Enable web connection
`sudo ufw allow www`
`sudo ufw status`
`sudo ufw allow 443/tcp`
* Mysql
`sudo mysql_secure_installation`
* Change the root password and confirm.
* Cleanup, delete unneeded things in vps:
`sudo rm -rf /etc/motd.tail`


### Initial Nginx Configuration
* Nginx Configuration File: `/etc/nginx/sites-available`
* Create domains and subdomains:
`/var/www/main`
`/var/www/api`
`/var/www/myaccount`
* Create index.php on each.
* Modify `/etc/nginx/sites-available/main`. :
```
  server { # php/fastcgi
    listen       80;
    listen       [::]:80;
    
    root         /var/www/main;
    index        index.php index.html;
    
    server_name  domain1.com www.domain1.com;
    access_log   logs/domain1.access.log  main;
    
    location ~ \.php$ {
      fastcgi_pass   127.0.0.1:1025;
      try_files $uri =404;
      fastcgi_pass_path_info ^(.+\.php)(/.+)$;
      fastcgi_pass unix:/var/run/php8.1-fpm.sock;
      fastcgi_index index.php;
      include fastcgi_params;
    }
    
    location ~ /\.ht {
        deny all;
    }
  }
```
* Create a symlink: `sudo ln -s /etc/nginx/sites-available/main /etc/nginx/sites-enabled/main`
* Check contents: `ls -a /etc/nginx/sites-enabled/`
* Check config file syntax of nginx: `sudo nginx -t` then reload `sudo nginx reload`
* Do the same for the api amd myaccount: `sudo cp main api` `sudo cp main myaccount`
* Modify subdomains on nginx config: `server_name  api.domain1.com www.api.domain1.com;` `server_name  myaccount.domain1.com www.myaccount.domain1.com;`
* Enabling and configure gzip on `/etc/nginx/nginx.conf`
```
    gzip on;
    ip_disable "msie6";
    
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain application/xml text/css application/json application/xjavascript text/xml application/xml+rss text/javascript;
```
`sudo service nginx reload`
* Establish Expiration Headers

## Support

<a href="https://www.buymeacoffee.com/pristineweb" target="_blank"><img src="https://www.buymeacoffee.com/assets/img/custom_images/purple_img.png" alt="Buy Me A Coffee" style="height: 41px !important;width: 174px !important;box-shadow: 0px 3px 2px 0px rgba(190, 190, 190, 0.5) !important;-webkit-box-shadow: 0px 3px 2px 0px rgba(190, 190, 190, 0.5) !important;" ></a>

## License

MIT

---

> GitHub [@william](https://github.com/william251082)

