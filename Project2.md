# LEMP means NGINX Mysql PHP

 ## 1.  **Linux**

> I made use of VMWare to install an ubuntus server on my local system.

To update the OS repositories, I ran the following codes

```bash
sudo apt update
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/update.png)



## 2. **Installing Nginx**

To Install the nginx (web) server, I ran the following codes

```bash
sudo apt install nginx -ys
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/InstallNginx.png)

And I ran the following code to verify the nginx installation 

```bash
which nginx
```
```bash
sudo systemctl status nginx
```
> To verify the status of the nginx server



![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/WhichNginx.png)

To verify if the nginx server is up and running, i just grabbed the IP address of the Linux system by running the `ip a` command and put it in the browser and the (default) page below is displayed or by running the `curl localhost` command to the the same result in text format.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/VerifyNginx.png)

## 3. **Installing Mysql**

To install a mysql-server which will serve as the database of the stack, I ran the following codes

```bash
sudo apt install mysql-server
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/InstallMysql.png)

To verify mysql-server is running and to change the password for the root user:

```bash
sudo mysql 
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Password@1';
exit
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/MysqlConfig.png)

For additional security config, i ran the command below, and responded to the prompts as neccessary

```bash
sudo mysql_secure_installation
```

## 4. **Installing & Configuring PHP**
To install PHP and all dependencies for both  mysql and Apache, I ran the following codes

```bash
sudo apt install php-fpm php-mysql
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/InstallPHP.png)

And when prompted, I hit the Y button and pressed ENTER to confirm installation.

>I ran a sequence of codes to do the following:

1. create the root web directory for projectLEMP.
2. Change the ownership of the created directory.
3. Open a new configuration file in Nginx’s sites-available directory using the nano command-line editor.

```bash
sudo mkdir /var/www/projectLEMP 
```
```bash
 sudo chown -R $USER:$USER /var/www/projectLEMP
```
```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/codes.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/LEMPConfigFile.png)
> I populated the `projectLEMP` file with the text above.

>I also ran another batch of codes to do the following:
1. Activate my configuration by linking to the config file from Nginx’s sites-enabled directory,
2. Test my configuration for syntax errors,
3. Disable default Nginx host that is currently configured to listen on port 80,
4. Reload Nginx to apply the changes.


I also confirmed the creation of the configuration file by running:

```bash
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```
> This links my configuration to nginx's `sites-enabled` directory. This will tell Nginx to use the configuration next time it is reloaded.
```bash
sudo nginx -t
```
>This should return a "...syntax is ok" message if all went well.
```bash
sudo unlink /etc/nginx/sites-enabled/default
```
> To disable default Nginx host that is currently configured to listen on port 80.


```bash
sudo systemctl reload nginx
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/codes.png)

I created a simple index.html file to serve as the root of the new website by running:

```bash
sudo echo 'Hello LEMP from localhost'  > /var/www/projectLEMP/index.html

```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/NewIndexFile.png)

I then verified if the websites are served by the new web directory by inpuuting the Public IP address in a browser, and the result was the page below, which is a graphical representaion of the simple index.html that was written. 

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/NewWebPage.png) 

>This shows that websites are being served by the projectLEMP web root directory.

## 5. **Testing PHP with Nginx**

To test if the nginx can correctly serve `.php` files, I created a test PHP file in the document root,

```bash
sudo nano /var/www/projectLEMP/info.php
```
and inserting the following text into the file.

```bash
<?php
phpinfo();
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/info.PHP.png) 

To verify this, I added the  __*/info.php*__ suffix to the Public IP address of the Linux system.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/PhpVerfied.png) 

>The result is a web page containing detailed information about the server as shown above.

As the page generated contains sensitive information about the PHP server, it was deleted after reviewing the information on it by running the command below.


```bash
sudo rm /var/www/projectLEMP/info.php
```

## 5. **Retrieving Data from Mysql Database With PHP**

 I connected to the Mysql console by running the command
 ```bash
 sudo mysql -p
 ``` 
 I then went ahead to supply the password set for the root user of the Mysql console.
 
 I then ran a series of command to achieve the following:
 
 1. Create a Database named sampleDB,
 1. Create a new user called samleUSER, and 
 1. give this sampleUSER permission over the sampleDB.

  
 
  
<!--   
  ```bash
 mysql> CREATE DATABASE sampleDB;
 ``` 
  ```bash
 mysql> CREATE USER 'sampleUSER' IDENTIFIED WITH mysql_native_password BY 'P@ssword1';
 ```  -->
 
  ```bash
 mysql> CREATE DATABASE sampleDB;
 ``` 
 ```bash
 mysql> CREATE USER 'sampleUSER' IDENTIFIED WITH mysql_native_password BY 'P@ssword1';
 ```
 ```bash
 mysql> GRANT ALL ON sampleDB.* TO 'sampleUSER';
 ```
  ```bash
 mysql> exit
 ``` 

 To login to the Mysql console as the newly created user, i ran the code below.

 ```bash
 mysql -u sampleUSER -p
 ```
   ```bash
 mysql> SHOW DATABASES;
 ```
 > I then put in the password used when creating the user when promted.

 
 ![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/ShowDBs.png)

 Next, I created a test table named *todo_list*. From the MySQL console by running the following statement:

```
CREATE TABLE sampleDB.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/CreateTable.png)


I inserted a few rows of content in the test table. by running the **INSERT** command a few times using different values


```bash
mysql>  INSERT INTO sampleDB.todo_list (content) VALUES ("My first important item");
mysql>  INSERT INTO sampleDB.todo_list (content) VALUES ("My second item on the list");
mysql>  INSERT INTO sampleDB.todo_list (content) VALUES ("The third item on the list is a charm");
```

To confirm that the data was successfully saved to the table, I ran:

```bash
mysql>  SELECT * FROM sampleDB.todo_list;
```
> And the output was as shown below.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/Write2D.png)

After validating entries to the database, i exited the mysql console.

Using nano text editor, I then created a `.php` script that will connect to MySQL and query for content. 

```bash
vi /var/www/projectLEMP/todo_list.php
```

I created this  new PHP file in my custom web root directory and saved the script below in it:

```bash
<?php
$user = "sampleUSER";
$password = "P@ssword1";
$database = "sampleDB";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
Finally i grabbed the  IP address of the Linux system again and append */todo_list.php* as a suffix, and the browser returned the content of the todo_list Table in the sampleDB earlier created as shown below.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project2/WebViewofDB.png)
