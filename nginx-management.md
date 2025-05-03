# Nginx Management.

## Installing Nginx.

Here’s how to perform a **clean installation of Nginx** on Ubuntu:

### Step 1: Update Your Package List
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Nginx
```bash
sudo apt install nginx -y
```

### Step 3: Enable and Start the Nginx Service
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Step 4: Check Nginx Status
```bash
sudo systemctl status nginx
```

You should see it **active (running)**.

### Step 5: Verify Installation in Browser or CLI

- Open a browser and visit:  
  ```
  http://your-server-public-ip
  ```
  You should see the **default Nginx welcome page**.

- Or use `curl`:
  ```bash
  curl http://localhost
  ```

Would you like help setting up a custom server block (virtual host) now?

### Adding SSl and setting up for reverse proxy-ing.

1. Open nginx config file.

```bash
sudo vim /etc/nginx/nginx.conf
```

2. Add the block below, and implement as written in the comment.

```shell
#include /etc/nginx/sites-enabled/*; - this line does not need to be added if already available - simply add the hashtag to comment it Ensure it is commented

server {
    listen      80;
    listen   [::]:80;
    server_name api.staging.kingofpimall.com;

    location / {
        proxy_pass http://localhost:8080/;
    }
}
```

2. Configure SSL with certbot and let's encrypt.

```bash
sudo apt update

sudo apt install certbot python3-certbot-nginx

sudo certbot --nginx -d api.zedlabs.xyz
```

**a sample progress response**

```bash
ubuntu@ip-172-31-39-118:~$ sudo certbot --nginx -d api.zedlabs.xyz
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Certificate not yet due for renewal

You have an existing certificate that has exactly the same domains or certificate name you requested and isn't close to expiry.
(ref: /etc/letsencrypt/renewal/api.zedlabs.xyz.conf)

What would you like to do?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: Attempt to reinstall this existing certificate
2: Renew & replace the certificate (may be subject to CA rate limits)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
```

> In my case, I decided to renew, so I entered '2', and it was successful.

**a sample showing further progress and success response**

```bash
Renewing an existing certificate for api.zedlabs.xyz

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/api.zedlabs.xyz/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/api.zedlabs.xyz/privkey.pem
This certificate expires on 2025-07-30.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for api.zedlabs.xyz to /etc/nginx/nginx.conf
Your existing certificate has been successfully renewed, and the new certificate has been installed.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

**another sample progress and final success sample - for creating a fresh certificate**

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): hello.zedlabs@gmail.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.5-February-24-2025.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Account registered.
Requesting a certificate for jenkins.zedlabs.xyz

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/jenkins.zedlabs.xyz/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/jenkins.zedlabs.xyz/privkey.pem
This certificate expires on 2025-07-30.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for jenkins.zedlabs.xyz to /etc/nginx/nginx.conf
Congratulations! You have successfully enabled HTTPS on https://jenkins.zedlabs.xyz

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

**if you get the any error regarding the domain not being found(in the server block), that means you failed to add the domain as done in the step 1 of this section. Proceed to update the server name from "_" to as below - as done in the server block above**

```bash
server_name stage.api.terabyte.africa;
```

run the command again

```bash
sudo certbot --nginx -d stage.api.terabyte.africa
```

3. restart nginx

```bash
sudo systemctl restart nginx
```

4. then restart you app service

```bash
sudo systemctl restart Terabyte-backend-test-server.service
```

## Removing/Uninstalling Nginx.

### Step 1: Stop the Nginx Service
```bash
sudo systemctl stop nginx
```

### Step 2: Uninstall Nginx

#### For just the main package:

```bash
sudo apt remove nginx
```

#### For a full cleanup (including config files):
```bash
sudo apt purge nginx nginx-common
```

### Step 3: Remove Any Leftover Files (Optional)

If you want to delete everything Nginx-related, including custom config files or logs:

```bash
sudo rm -rf /etc/nginx /var/log/nginx /var/www/html
```

### Step 4: Autoremove Unused Dependencies

```bash
sudo apt autoremove
```

You can verify that it's uninstalled with:
```bash
which nginx
```
It should return nothing if Nginx is fully removed.

**final sample of an nginx config file after reverse proxy and ssl**

```bash
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        # include /etc/nginx/sites-enabled/*;

        server {
                server_name api.zedlabs.xyz;

         location / {
                proxy_pass http://localhost:5000/;
           }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.zedlabs.xyz/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.zedlabs.xyz/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}


        server {
    if ($host = api.zedlabs.xyz) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


                listen      80;
                listen   [::]:80;
                server_name api.zedlabs.xyz;
    return 404; # managed by Certbot
}}

#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}
```