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
* [Nginx Advanced Security Configuration](#nginx-advanced-security-configuration)
* [Deploy an App](#deploy-an-app)
* [Setting HTTPS/TLS Websites Using LetsEncrypt](#setting-an-app)
* [Upgrading OS](#upgrading-os)

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
    
    server_name  <your_domain>.com www.<your_domain>.com;
    access_log   logs/<your_domain>.access.log  main;
    
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
* Establish Expiration Headers `/etc/nginx/nginx.conf`
```
server {
    location ~* .(jpg|jpeg|png|gif|ico|css|js|eot|svg|ttf|woff|pdf)$ {
        expires 365d;
    }
    return 301 $scheme://<your_domain>.com;
}
```

### Nginx Advanced Security Configuration
* Solve Privacy problems
* `/etc/nginx/nginx.conf`
`sudo service nginx reload`
* `server_tokens off;` Hide nginx version on response headers.
`/etc/php8.1/fpm/php.ini`
* `expose_php = Off;` Hide php and os version on response headers.
`sudo service php8.1-fpm reload`
* Put to this to all nginx config files: `sudo nano /etc/nginx/sites-available/*`
```
    root         /var/www/main;
    index        index.php index.html;
    
    fastcgi_hide_headers 'X-Powered-By'
```
`sudo service nginx reload`
* Avoiding XSS Attacks on Nginx
`/etc/nginx/nginx.conf`
```
    # Avoiding XSS Attacks
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
```
* Mitigating DoS and DDoS Attacks
benchmark: `ab -t 30 -c 10 https://<your_domain>.com`
```
    # Avoiding XSS Attacks
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    
    # Mitigating DoS and DDoS Attacks
    client_body_buffer_size 4m;
    large_client_header_buffers 4 4m;
    limit_conn_zone $binary_remote_addr zone=req_limit_per_ip:10m rate=3r/s;
    limit_conn conn_limit_per_ip 15;
    limit_req zone=req_limit_per_ip burst=20;
```

### Deploy an App
* /var/www/ git clone <app_github_url> main
* Install all dependencies: `composer install`
* Configure DB 
`mysql -root -p`
`create <db_name>`
* configure the .env file, put all needed values.
`php artisan key:generate` -- generate app key
* Execute migration 
`php artisan migrate`
`php artisan migrate:refresh`
* Configure Nginx
```
    root /var/www/main/public;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
```
* Give the right folder permission:
`sudo chown -R www-data storage/`
`sudo chown -R www-data bootstrap/cache/`
`sudo chown -R www-data public/`

### Setting HTTPS/TLS Websites Using LetsEncrypt
* Obtaining and Installing LetsEncrypt
* `git clone https://github.com/acmesh-official/acme.sh.git`
* `cd ./acme.sh`
* `./acme.sh --install -m my@example.com`
* `crontab -e`
* For cron to check and renew if needed:
* `0 0 * * * "/home/user/.acme.sh"/acme.sh --cron --home "/home/user/.acme.sh" > /dev/null`
* Obtaining Free Certificates, For Every Domain, With LetsEncrypt.
* `root@v1:~# acme.sh -h`
* `acme.sh --issue -d <your_domain>.com -w /var/www/main/public` 
* `acme.sh --issue -d api.<your_domain>.com -w /var/www/api/public` 
* `acme.sh --issue -d myaccount.<your_domain>.com -w /var/www/myaccount/public` 
* le folder: `cd .le/`
* Installing the Certificates To Nginx
* Nginx ssl config generator https://ssl-config.mozilla.org/
* `nginx -v`
* `openssl`
* `<OpenSSL> version`
  `sudo nano /etc/nginx/sites-available/main`
```
server {
    listen 80;
    listen [::]:80
    
    server_name  <your_domain>.com www.<your_domain>.com;
    
    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    return 301 https://<your_domain>$request_uri;
}
```
* Copy and paste the generated ssl:
```
    # generated 2022-09-26, Mozilla Guideline v5.6, nginx 1.17.7, OpenSSL 1.1.1k, modern configuration
# https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=modern&openssl=1.1.1k&guideline=5.6
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /path/to/signed_cert_plus_intermediates; // modify this to your own
    ssl_certificate_key /path/to/private_key; // modify this to your own
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # modern configuration
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;
    
    ssl_dhparam your_path/.le/dhparams.pem  // modify this to your own

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates; // modify this to your own

    # replace with the IP address of your resolver
    resolver 127.0.0.1;
}
```
* Do the same on api and myaccount subdomains.
* Improving the Security with dhparams. https://www.ssllabs.com/ssltest/
* `openssl dhparam -out dhparams.pem 2048`
* .le/dhparams.pem should exist.
* Install fail2Ban to avoid brute force attacks
* `sudo apt-get install fail2ban`
* Copy config then modify
* `cp jail.conf jail.local`
* `/etc/fail2ban`
```
    bantime = 600
    findtime = 600
    maxretry = 5
```

### Upgrading OS
* Make a snapshot
* Update all packages `apt-get upgrade` `apt-get dist-upgrade`
* `do-release-upgrade`
* additional ssh daemon port 1022
* Open a new connection using this: `iptable -I INPUT -p tcp --dport 1022 -j ACCEPT`
* Boot the OS
* Resolve nginx.conf: show diff `D`

## Support

<a href="https://www.buymeacoffee.com/pristineweb" target="_blank"><img src="https://www.buymeacoffee.com/assets/img/custom_images/purple_img.png" alt="Buy Me A Coffee" style="height: 41px !important;width: 174px !important;box-shadow: 0px 3px 2px 0px rgba(190, 190, 190, 0.5) !important;-webkit-box-shadow: 0px 3px 2px 0px rgba(190, 190, 190, 0.5) !important;" ></a>

## License

MIT

---

> GitHub [@william](https://github.com/william251082)
