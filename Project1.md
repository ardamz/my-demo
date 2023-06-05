# LAMP means Linux Apache Mysql PHP

 ## 1.  **Linux**

> I made use of VMWare to install an ubuntus server on my local system.

To update the OS repositories, I ran the following codes

```bash
sudo apt update
```
![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/UpdateRepos.png)

## 2. **Apache**

To Install the Apache (web) server, I ran the following codes

```bash
sudo apt install apache2 -y
```

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/install%20apache2.png)

And I ran the following code to verify the Apache installation and status

```bash
curl localhost
```
To verify if the Apache server is up and running, i just grab the IP address of the Linux system and put it in the browser and the (default) page below is displayed

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/ItWorks.png)

## 3. **Mysql**

To install a mysql-server which will serve as the database of the stack, I ran the following codes

```bash
sudo apt install mysql-server -y
```
![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/install%20mysql.png)

To verify mysql-server is running and to change the password for the root user:

```bash
sudo mysql 
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Password@1';
exit
```

For additional security config, run the command below, and respond to the prompts as neccessary

```bash
sudo mysql_secure_installation
```

## 4. **PHP**
To install PHP and all dependencies for both  mysql and Apache, I ran the following codes

```bash
sudo apt install php libapache2-mod-php php-mysql
```
![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/InstallPHP.png)

And to verify if PHP has installed properly, run the follwing code to check the verson of PHP insatlled

```bash
php -v
```

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/C:\Users\gbeng\OneDrive\Documents\git\my-demo\project1\PHPverification.png.png)

I created the directory for projectlamp and changed the owner by running following commands:

```bash
sudo mkdir /var/www/projectlamp 
```
```bash
 sudo chown -R $USER:$USER /var/www/projectlamp
```

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/projectlamp%20dr%20created%20ownership%20changed.png)

I created a new configuration file in Apache’s sites-available directory using `vi`:

```bash
sudo vi /etc/apache2/sites-available/projectlamp.conf
```

Then i inserted the following text into the configuration file using the NANO text editor

```bash
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/using%20nano%20to%20create%20the%20config%20file.png)

I also confirmed the creation of the configuration file by running:

```bash
sudo ls /etc/apache2/sites-available
```

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/projectlamp%20cofig%20file%20confirmed.png)

I then ran a series of commands to inform apache to:
1. serve projectlamp using /var/www/projectlampl as its web root directory.
1. disable the default website that comes installed with Apache.
1. To make sure your configuration file doesn’t contain syntax errors, and
1. Finally, reload Apache so these changes take effect.

```bash
sudo a2ensite projectlamp
```

```bash
sudo a2dissite 000-default
```

```bash
sudo apache2ctl configtest
```

```bash
sudo systemctl reload apache2
```

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/server%20cofigured.png)

I created a simple index.html file to serve as the root of the new website by running:

```bash
sudo echo 'Hello LAMP from localhost' > /var/www/projectlamp/index.html
```

I then verified if the websites are served by the new web directory by inpuuting the Public IP address in a browser, and the result was the page below, which is a graphical representaion of the simple index.html that was written. 

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/BrowserVerification.png)

tToo test the PHP on the website, i changed the order of the PHP configuration file by editing the file using the Vim text editor:

```bash
sudo vim /etc/apache2/mods-enabled/dir.conf
```

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/apache2%20defaults%20altered.png)

Then i ran the following codes to 
1. reload the Apache server and 
```bash
sudo systemctl reload apache2
```
2. create a new file named index.php inside your custom web root folder.
```bash
vim /var/www/projectlamp/index.php
```
3. populate the file above with a simple and valid php code
```bash
<?php
phpinfo();
```
I then refreshed the website in my browser, and i got page below which provides information about the  server from the perspective of PHP

![Screenshot](https://github.com/ardamz/PersonalDemos/blob/main/1.%20Project%201%20LAMP%20Stack%20Implementation/PHP%20verified.png)

 I ran the command below to remove the created php file as it contains sensitive information, and it can easily be recreated in the future if needed.

 ```bash
sudo rm /var/www/projectlamp/index.php
 ```
