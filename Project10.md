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

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/NginxConfig.png)

3. To setup my domain, I took the following steps;
                A. Registered the `ardamz.uk` domain.
                B. I created a hosted zone for `ardamz.uk` in AWS Route 53.
                C. I updated the name servers for `ardamz.uk` with those from AWS Route 53
                D. In the hosted zone, i created A records for `ardamz.uk` and `www.ardamz.com` both pointing to the public IP address of my Load Balancer

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/JenkinsHome.png)

3. I performed the initial setup by first getting the Admin password by running 
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
then i installed plugins, created an admin user and was done with the inital setup.

>To effect this changes, I restarted the apache service `sudo systemctl restart apache2`


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/AddUser.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/SetupComplete.png)

## STEP 2.  **Configure Jenkins to retrieve source codes from GitHub using Webhooks**

1. logged into my github account and went into the settings tab for this projects repository and selected the webhook option from the menus on the left the clicked the `Add Webhook` button. I put in the public IP Address of the jenkins server in the payload url, changed the `content type` and left every other setting as default.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Webhook.png)


2. Logged into my jenkins server as tthe admin user, then from the dashboard I selected `Create a job` put in a project name, selected the `Fresstyle Project` option and the pressed the OK button.

3. On the configuration page, I supplied the the github repository (didn't supply the credential as the repository i was using was set as public and not private.), selected a build trigger and post_build action as shown below.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/GitRepo.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/BuildTrigger.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/PostBuild.png)

4. After the configutation, clicking on the `Build Now` button increases the build history of the project. This shows that the project configuration is okay, the git configutation supplied in step 3 above was to enable an automatic triggering of the build process (any commit on the github repo, triggers a build process in jenkins).
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/AutoBuild.png)

## STEP 3.  **Configure Jenkins to copy files to NFS server via SSH**

1. From the Jenkins Dashboard>Manage Jenkins>Plugins I installed (without restart) “Publish Over SSH” plugin from the list of available plugins.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/PublishOverSSH.png)

2. From the Jenkins Dashboard>Manage Jenkins>System I scrolled down to the `Publish over SSH` section and supplied the following details;

         A. Private key used to connect to the NFS Server

         B. Details of the NFS Server like username, hostname and remote directory.


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/ConfigureSSH.png)

3. After saving the SSH configuration, I went to my configuration page for my project Dashboard>project9>Configuration, and added `Send build artifacts over SSH` from the list of post-build action. I inserted `**` in the place of Source files to iindicate all files.

4. Saved the new post-build action and went to make an update in my git repo, and the build was initiated in my jenkins enviroment automatically. I kept getting a `Permission denied` Error

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/BuildErrors.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Build8.png)

5. I modified the permission and ownership of the /mnt/apps directory

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Permission.png)

6. After modifing the permissions I was able to get a succesful build after a push action from my git repo. I also verified this successful build by checking the /var/www folders of my my web servers (as they are mounted on the /mnt/apps dircetory of the NFS Server).

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/Build9.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project10/CopyVerified.png)