# Load Balancer Solution With Apachegit stat
# Load Balancer Solution With Apache



1. Still maintaining my infrastructure from the last project, I created a new ubuntu ec2 instance named ` Project-8-apache-lb`, and ran the following commands;
 ```bash 
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev -y

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2

#Confirm Apache is running
sudo systemctl status apache2
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project8/update.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project8/InstallApache.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project8/ApacheStatus.png)

2. I updated the apache config file by running.
 ```bash 
sudo vi /etc/apache2/sites-available/000-default.conf
```
and inserted the following lines of code

```bash
<Proxy "balancer://mycluster">
               BalancerMember http://172.31.84.98:80 loadfactor=5 timeout=1
               BalancerMember http://172.31.89.6:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```
>To effect this changes, I restared the apache service `sudo systemctl restart apache2`
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project8/ApacheConfig.png)
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project8/ApacheConfig1.png)

3. I verified my setup thus far by putting the public IP address of the LoadBalancer in the browser and voila!!!
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project8/LB-PublicIP.png)


4. On each of the web-servers, I unmounted the logs directory from the `/mnt/logs` directory on the NFS Server by running;
```bash
sudo umount -l /var/log/httpd
```
> used the -l as it was returning a mount point busy error.

5. I restarted both servers and anytime i refreshed webpage of the loadbalancer  I was able to read their access logs (increasing in turns) by running 
```bash
sudo tail -f /var/log/httpd/access_log
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project8/LocalLogs.png)
