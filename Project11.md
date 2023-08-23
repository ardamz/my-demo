# Ansible – Automate Project 7 to 10

## STEP 1.  **Install and Configure Ansible on EC2 Instance**

1. Still maintaining my infrastructure from the last project, I changed the name of the `Jenkins Server`  to `Jenkins-Ansible`. This will be use to run the ansible playbooks.

2. I installed ansible on the server and checked the version of ansible installed by running the following lines of code;

 ```bash 
sudo apt update
sudo apt install ansible -y

# Check version of Ansible
ansible --version
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/InstallAnsible.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/AnsibleVersion.png)

3. In my `guthub` account, I created a new repositiry named `ansible-config-mgt`.
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

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/update.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/InstallAnsible.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/AnsibleVersion.png)

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
>To effect this changes, I restarted the apache service `sudo systemctl restart apache2`
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/ApacheConfig.png)
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/ApacheConfig1.png)

3. I verified my setup thus far by putting the public IP address of the LoadBalancer in the browser and voila!!!
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/LB-PublicIP.png)


4. On each of the web-servers, I unmounted the logs directory from the `/mnt/logs` directory on the NFS Server by running;
```bash
sudo umount -l /var/log/httpd
```
> used the -l as it was returning a mount point busy error.

5. I restarted both servers and anytime i refreshed webpage of the loadbalancer  I was able to read their access logs (increasing in turns) by running 
```bash
sudo tail -f /var/log/httpd/access_log
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/LocalLogs.png)

NOTES: This was an interesting one as i learnt of the various Load balancing comcepts and some scenarios where they can be deployed.
