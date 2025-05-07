# Nginx Management.

A detailed sheet with different PRACTICAL life-saving tutorials for working with Nginx on AWS virtual machines(VMs).

> Nginx is a proxy - a reverse proxy. It sits in front of your server, intercepts every http request to your
> server, and helps as lot to improve security. It also provides massive support for reverse-proxying, caching, load-balancing,
> and a lot more I guess. [Read more](https://nginx.org).


## Installing Nginx.

Here’s how to perform a **clean installation of Nginx** on Ubuntu - my preferred OS for VMs:

1. Update Your Package List

```bash
sudo apt update && sudo apt upgrade -y
```

2. Install Nginx

```bash
sudo apt install nginx -y
```

3. Enable and Start the Nginx Service

```bash
# sudo systemctl daemon-reexec
# sudo systemctl daemon-reload
sudo systemctl enable nginx
sudo systemctl start nginx
# sudo systemctl restart nginx
# sudo systemctl stop nginx
```

4. Check Nginx Status

```bash
sudo systemctl status nginx
```

You should see it **active (running)**.

5. Verify Installation in Browser or CLI

- Open a browser and visit:  

```bash
http://your-server-public-ip # as earlier stated, Nginx would intercept all incoming http traffic - the nginx home-screen will show
```
You should see the **default Nginx welcome page**.

- Or use `curl`:

```bash
curl http://localhost
```

### Adding SSl to Domains, and Setting up for Reverse Proxy-ing.

1. Open nginx config file.

```bash
sudo vim /etc/nginx/nginx.conf
```

or(preferably)

```bash
nano nano /etc/nginx/nginx.conf
```

2. Add the block below, and implement as written in the comment.

```bash
# include /etc/nginx/sites-enabled/*; - this line does not need to be added if already available - simply add the hashtag to comment it. Ensure it is commented

server {
    listen      80;
    listen   [::]:80;
    server_name domain_or_sub_domain; # e.g. api.zedlabs.xyz, jenkins.zedlabs.xyz, grafana.zedlabs.xyz, or just zedlabs.xyz

    location / {
        proxy_pass http://localhost:port/; # e.g https:localhost:8080
    }
}
```

With the above set-up, you're able to auto-redirect Nginx to serve your project straight-up, while also preparing to
get a free SSL certificate for your domain with only some few extra commands as can be seen below.

3. Configure SSL with certbot and Let's Encrypt.

```bash
sudo apt update

sudo apt install certbot python3-certbot-nginx

sudo certbot --nginx -d domain_or_sub_domain # e.g. api.zedlabs.xyz
```

**a sample request and progress response - on attempting to renew an SSL cert**

```bash
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

**a sample progress and final success sample - for creating a fresh certificate**

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

**if you get any error regarding the domain not being found(in the server block), that means you failed to add the domain as done in the step 1 of this section. Proceed to update the server name from "_" to as below - as done in the server block above**

```bash
server_name domain_or_sub_domain;
```

run the command again

```bash
sudo certbot --nginx -d domain_or_sub_domain
```

4. restart nginx

```bash
sudo systemctl restart nginx
```

<!-- 5. then restart you app service

```bash
sudo systemctl restart application-service-name.service
``` -->

## Removing/Uninstalling Nginx.

1. Stop the Nginx Service

```bash
sudo systemctl stop nginx
```

2. Uninstall Nginx

- For just the main package:

```bash
sudo apt remove nginx
```

- For a full cleanup (including config files):

```bash
sudo apt purge nginx nginx-common
```

3. Remove Any Leftover Files (Optional)

If you want to delete everything Nginx-related, including custom config files or logs:

```bash
sudo rm -rf /etc/nginx /var/log/nginx /var/www/html
```

4. Auto-remove Unused Dependencies

```bash
sudo apt autoremove
```

5. Verify Uninstallation

```bash
which nginx
```
It should return nothing if Nginx is fully removed.