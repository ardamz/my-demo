# Continous Integration Pipeline For Tooling Website

 ## STEP 1.  **Provision Jenkins Server**


1. I created an EC2 instance based on Ubuntun 22.04 adn named it `Jenkins`, to install Jenkins, I ran the following commands;
 ```bash 
 # Install JDK
sudo apt update
sudo apt install default-jdk-headless -y

# Install Jenkins:
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

#Confirm Jenkins is running
sudo systemctl status jenkins
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/update.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/InstallJenkins.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/JenkinsStatus.png)

2. I confirmed the installation by pasting the public IP address of the `Jenkins Server` in a web brwoser along with the default port (8080) which Jenkins uses.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/JenkinsHome.png)

3. I performed the initial setup by first getting the Admin password by running 
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
then i installed plugins, created an admin user and was done with the inital setup.

>To effect this changes, I restarted the apache service `sudo systemctl restart apache2`


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/AddUser.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/SetupComplete.png)

## STEP 2.  **Configure Jenkins to retrieve source codes from GitHub using Webhooks**

1. logged into my github account and went into the settings tab for this projects repository and selected the webhook option from the menus on the left the clicked the `Add Webhook` button. I put in the public IP Address of the jenkins server in the payload url, changed the `content type` and left every other setting as default.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/Webhook.png)


2. Logged into my jenkins server as tthe admin user, then from the dashboard I selected `Create a job` put in a project name, selected the `Fresstyle Project` option and the pressed the OK button.

3. On the configuration page, I supplied the the github repository (didn't supply the credential as the repository i was using was set as public and not private.), selected a build trigger and post_build action as shown below.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/GitRepo.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/BuildTrigger.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/PostBuild.png)

4. After the configutation, clicking on the `Build Now` button increases the build history of the project. This shows that the project configuration is okay, the git configutation supplied in step 3 above was to enable an automatic triggering of the build process (any commit on the github repo, triggers a build process in jenkins).
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/AutoBuild.png)

## STEP 3.  **Configure Jenkins to copy files to NFS server via SSH**

1. From the Jenkins Dashboard>Manage Jenkins>Plugins I installed (without restart) “Publish Over SSH” plugin from the list of available plugins.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/PublishOverSSH.png)

2. From the Jenkins Dashboard>Manage Jenkins>System I scrolled down to the `Publish over SSH` section and supplied the following details;

         A. Private key used to connect to the NFS Server

         B. Details of the NFS Server like username, hostname and remote directory.


![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/ConfigureSSH.png)

3. After saving the SSH configuration, I went to my configuration page for my project Dashboard>project9>Configuration, and added `Send build artifacts over SSH` from the list of post-build action. I inserted `**` in the place of Source files to iindicate all files.

4. Saved the new post-build action and went to make an update in my git repo, and the build was initiated in my jenkins enviroment automatically. I kept getting a `Permission denied` Error

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/Errors.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/Build8.png)

5. I modified the permission and ownership of the /mnt/apps directory

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/Permission.png)

6. After modifing the permissions I was able to get a succesful build after a push action from my git repo. I also verified this successful build by checking the /var/www folders of my my web servers (as they are mounted on the /mnt/apps dircetory of the NFS Server).

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/Build9.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project9/CopyVerified.png)

