# Load Balancer Solution With Nginx and SSL/TLS

 ## STEP 1.  **Configure Nginx as a Load Balancer**


1. I deleted the Apache from my project 8 load balancer server and installed Nginx on it.

 ```bash 
 # Uninstall Apache and Install Nginx
sudo apt remove apache2
sudo apt install nginx -y
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/InstallNginx.png)

2. The `/etc/hosts` file remained unchanged from the previous installation, hence am able to reference their mnames in the nginx config file.

```bash
cat /etc/hosts

sudo vi /etc/nginx/nginx.conf
# insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name ardamz.uk www.ardamz.uk;
    location / {
      proxy_pass http://myproject;
    }
  }

# I commented out this line
#       include /etc/nginx/sites-enabled/*;

# Restart the Nginx Server and confirm its status
sudo systemctl restart nginx
sudo systemctl status nginx
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Hosts.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/NginxConfig.png)

3. To setup my domain, I took the following steps;
    *  Registered the `ardamz.uk` domain.
    -  I created a hosted zone for `ardamz.uk` in AWS Route 53.
    *  I updated the name servers for `ardamz.uk` with those from AWS Route 53
    -  In the hosted zone, I created A records for `ardamz.uk` and `www.ardamz.uk` both pointing to the public IP address of my Load Balancer

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Route53.png)

4. I installed certbot using the `snap` module by running the following commands

 ```bash 
 # Ensure snapd is running
sudo systemctl status snapd

 # Install Certbot
sudo snap install --classic certbot

 # Request my certifcate
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
> The request option looks up the domain from the `nginx` config file, and my domain was defined in the server block of my config file.
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/InstallCertbot.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/RequestSSL.png)

5. Accessing both domains on a web browser now show them being delivered using the secured https protocol.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Before.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Https.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Certificate.png)
