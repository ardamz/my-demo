# DevOps Tooling Website Solution

 ## A.  **Preparing The NFS Server**

> I utilised AWS EC2 as my server for this project. 

1. I created 3 additional volumes and attched them to the NFS server.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/Volumes.png)


2. I used the `gdisk` utility to create a single partition on each of the 3 disks for  the NFS Web Server.
 ```bash 
sudo gdisk /dev/xvdf
```
3. I ran the `lsblk` command to view the newly configured partition on each of the 3 disks on the server.
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/NewPartition.png)


4. I installed the `lvm2` utility by running;
```bash
sudo yum install lvm2 -y
```
I then checked for avalable partitons by running;
```bash 
sudo lvmdiskscan
```

5.  I used the `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM, by running   
```bash
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
```

> I verified creation PVs by running ` sudo pvs`

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/PhysicalVolumes.png)

6.  I used the `vgcreate` utility to add all 3 PVs to a volume group (VG) called `nfsdata-vg`

```bash
sudo vgcreate nfsdata-vg /dev/xvdh1  /dev/xvdg1 /dev/xvdf1
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/VGCreate.png)

7. On the NFS Server, I used the `lvcreate` utility to create 3 logical volumes. apps-lv, opt-lv and logs-lv of equal size from the `nfsdata-vg`. 

```bash
sudo lvcreate -n lv-apps -L 9.7G nfsdata-vg
sudo lvcreate -n lv-logs -L 9.7G nfsdata-vg
sudo lvcreate -n lv-opt -L 9.7G nfsdata-vg
```
<!---
opt-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
-->


[apps-lv and logs-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.]: 

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/lvcreate.png)
> I verified creation LVs by running ` sudo lvs`

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/sudoLVS.png)
>  had to edit the host name for easier identification.


8. I verfied my setup so far by running `sudo lsblk`

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/FinalSetup.png)
>looking good...

9. using the `mkfs` utility, I formated the Logical Volumes (LVs) on the NFS server with the `xfs` filsesystem.

```bash
sudo mkfs -t xfs /dev/nfsdata-vg/lv-apps
sudo mkfs -t xfs /dev/nfsdata-vg/lv-logs
sudo mkfs -t xfs /dev/nfsdata-vg/lv-opt
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/FileSystem.png)


10. On the NFS Server, using the `mkdir` command I created the `/mnt/logs`, `/mnt/apps` and `/mnt/opt` directory to serve as mount points for the LVs created above

```bash
sudo mkdir /mnt/apps /mnt/logs /mnt/opt
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/mkdir.png)


11. On the NFS Server, I created mount points on `/mnt` directory for the logical volumes as follow:
Mount `lv-apps` on `/mnt/apps` – To be used by webservers
Mount `lv-logs` on `/mnt/logs` – To be used by webserver logs
Mount `lv-opt` on `/mnt/opt` – To be used by Jenkins server in Project 8

```bash
sudo mount /dev/nfsdata-vg/lv-apps /mnt/apps
sudo mount /dev/nfsdata-vg/lv-logs /mnt/logs
sudo mount /dev/nfsdata-vg/lv-opt /mnt/opt
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/Mount.png)

12. I then verified my setup by running the `df -hT` comand as shown below.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/dfh.png)



## B.  **Installing And Configure NFS Server**

1. I updated the repository and installed the NFS server by running;

```bash
sudo yum update
sudo yum install nfs-utils -y
```

2. I then configured it to start on reboot and make sure it is always up and running by executing the following commands;
```bash
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/nfs-status.png)

3. I then set up permission that will allow our Web servers to read, write and execute files on NFS:

```bash
sudo chown -R nobody: /mnt/apps /mnt/logs /mnt/opt

sudo chmod -R 777 /mnt/apps /mnt/logs /mnt/opt

sudo systemctl restart nfs-server.service
```

4. I configured access to NFS for clients within the same subnet by specifying the subnet CIDR range in the `/etc/exports` file, and running the following commands.
```bash
sudo vi /etc/exports
sudo exportfs -arv
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/exports.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/exportfs.png)



5. I got the port number being used by the NFS Server by running, and updated the servers Security Group on AWS accordingly

```bash
rpcinfo -p | grep nfs
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/nfs_port4SG.png)
> In order for NFS server to be accessible from the client, I also opened following ports: TCP 111, UDP 111, UDP 2049.


<!-- ![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/SG_rules.png) -->


## C.  **Installing And Configure DB Servers**



To install a mysql-server which will serve as the database of the stack, I ran the following codes

```bash
sudo apt install mysql-server -y
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/InstallMysql.png)

To verify mysql-server is running and to change the password for the root user:

```bash
sudo mysql 
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Password@1';

 mysql> CREATE DATABASE tooling;
 
 mysql> CREATE USER 'webaccess'@'172.31.16.0/20' IDENTIFIED WITH mysql_native_password BY 'password';
 
 mysql> GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.16.0/20';
 mysql> FLUSH PRIVILEGES;
 
 exit
 ```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/webaccess.png)

I then changed the `bind-address` in the my mysql config using the vim command,

```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/bind-address.png)




## D.  **Preparing the Web Servers**

1. I launched three (3) new EC2 instances,

2. I ran the codes below  to install NFS client (on all 3 web servers),

```bash
sudo yum install nfs-utils nfs4-acl-tools -y
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/utils.png)

3. I created a `/var/www`directory to serve as a mount point for the NFS servers export for apps.

```bash
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid 172.31.24.217:/mnt/apps /var/www
sudo mount -t nfs -o rw,nosuid 172.31.24.217:/mnt/logs /var/log/httpd
```
>the second command was actually executed after installing apache.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/mount_www.png)

4. I ran the `df -h`command to verify the mount point. To ensure the changes will persist after reboot, i updated the `/etc/fstab`file by adding the line below.

 `172.31.24.217:/mnt/apps /var/www nfs defaults 0 0`
 `172.31.24.217:/mnt/logs /var/log/httpd nfs defaults 0 0`

5. I installed Remi’s repository, Apache and PHP by running the following lines,

```bash
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y

sudo dnf module reset php -y

sudo dnf module enable php -y

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1
```

6. I enabled TCP port 80 web servers Security Group on AWS, ran the `sudo setenforce 0` command and edited the the following config file;

```bash
sudo vi /etc/sysconfig/selinux
```
 I set SELINUXTYPE=disabled, and then restarted httpd.

7. I put the public IP address of any/all of the web servers in my browser, and i was able to get the default apache webpage.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/default-page.png)

8. On the web-server, I cloned the `tooling` repository from my github by running the following command;

```bash
sudo dnf install git -y
git clone https://github.com/ardamz/tooling.git
```
Using the `cd`command, i was able to navigate to thee `tooling` directory and then copied the `html` folder to `/mnt/apps`;

```bash
sudo cp -R html /var/www
```
9. Refreshing the webpages from step 7 above, I got the content that was cloned from the tooling repository.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/new-page.png)


10. I installed mysql client on the web servers by running,

```bash
sudo dnf install mysql -y
```
11. I updated the website’s configuration (`/var/www/html/functions.php`) file to connect to the database.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/website-config.png)

12. I went to AWS, and updated the security group of the DB Server to allow MYSQL traffic from the webservers.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/db-access.png)

13. I then applied the `tooling-db.sql` script from the cloned git repo to the database by running the coomand below, ands supplying the password when prompted.

```bash
mysql -h '172.31.3.3' -u 'webaccess' -p tooling < tooling-db.sql
```

14. I reloaded the webpage for any of the webservers, and supplied the following credentials `username: admin`and `password: admin`.
>This credentials were included in the `tooling-db.sql` script.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/landing-page.png)

15. I created in MySQL a new admin user with `username: myuser` and `password: password` by running the following commands on any of the webservers

```bash
mysql -h '172.31.3.3' -u 'webaccess' -p tooling 
mysql> INSERT INTO `users` (
    ->   `id`, `username`, `password`, `email`, `user_type`, `status`)
    -> VALUES (
    ->   2, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'dare@dare.com', 'admin', '1'
    -> );
```

16. I reloaded the webpage for any of the webservers, and supplied the following credentials `username: myuser` and `password: password`, and i was able to login.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project7/myuser.png)















