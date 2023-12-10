# Ansible Refactoring & Static Assignments (Imports and Roles)

## STEP 1.  **Jenkins job enhancement**

1. Still maintaining my infrastructure from the last project, on my `Jenkins-Ansible` server, i created a directory called `ansible-config-artifact` and mdoified its permissions by running the following code;

```bash
# Create the directory
# The home directory wasnt working for me, had to improvise
sudo mkdir /var/lib/jenkins/ansible-config-artifact

# Modify its permissions
sudo chmod -R 0777 /var/lib/jenkins/ansible-config-artifact
```

 ![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/Mkdir.png)


3. On my Jenkins web cosole, I installed the `Copy Artifact` plugin.

4. I created a new project called `save_artifacts`, and this project will be triggered by the completion of my exisiting `ansible` project.


 ```bash 
# Tried everything i could to make to home diorectory work, but its not...i give up.
```

## STEP 2.  **Refactor Ansible code by importing other playbooks into site.ymlRefactor Ansible code by importing other playbooks into `site.yml`**












































3. In my `guthub` account, I created a new repositiry named `ansible-config-mgt`.

4. I configured `Jenkins` by perfor,ming the following steps;
    *  I created a new Freestyle project called `ansible` in Jenkins and pointed it to my `ansible-config-mgt` repository.
    -  I configured Webhook in GitHub and set webhook to trigger `ansible` build.
    *  I configured a Post-build job to save all (**) files.
    
5. I tested my setup by making some changes to the main branch and made sure that builds starts automatically and Jenkins saved the files (build artifacts) as shown below;

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/Artifacts.png)


## STEP 2.  **Prepare the Development Environment using Visual Studio Code (VSC)**

1. VSC was configured to connect to my `ansible-config-mgt` repository.

2. i then cloned the `ansible-config-mgt` repository to my Jenkins-AAnsible Instance.
```bash
git clone https://github.com/ardamz/ansible-config-mgt.git
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/GitClone.png)

## STEP 3.  **Begin Ansible Developement**

1. I created a new branch called `Prj_11` in my `ansible-config-mgt` repository.

```bash
# Checkout and create a branch called Prj_11
git checkout -b Prj_11

# Create both playbooks and inventory directories
mkdir playboks && mkdir inventory

# Create common.yml in playbooks directory
cd playboks && touch common.yml

```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/Branch.png)

2. I created `playbooks` (used to store all your playbook files) and `inventotory` (used to keep your hosts organised) directories.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/Directory.png)

3. Created a `common.yml` file in the `playbooks` directory.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/Common.png)

4. Within the inventory directory,I created an inventory file (.yml) for each environment (Development, Staging, Testing and Production) dev, staging, uat, and prod respectively.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/Inventories.png)


## STEP 4.  **Set up an Ansible Inventory**
For ansible to be able to login to the other servers, i needed to find a way to import the private key to the `Jenkins-Ansible` server, to solve this problem, I had to make use of an `SSH Agent.` To do this I did the following;

1. I exitted my `Jenkin-Ansible` server,
2. i ran the codes below

```bash
# Start and configure SSH Agent
eval `ssh-agent -s`

# Add Private key to the agent
ssh-add pbls2.pem

# Confirm key has been added
ssh-add -l

```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/SshAdd.png)

3. Access the `Jenkins-Ansible` server using ssh-agent by running

```bash
 ssh -A ubuntu@3.80.32.204
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/SshAgentLogin.png)

4. After setting up and verifying the ssh-agent, I updated the `inventory/dev.yml` file with the details of the various servers.


## STEP 5.  **Create A Common Playbook**

1. I populated the playbooks.common.yml with the following lines of code

```bash
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```





















 ```bash 
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev -y

#Confirm Apache is running
sudo systemctl status apache2
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/update.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/InstallAnsible.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/AnsibleVersion.png)

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
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/ApacheConfig.png)
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/ApacheConfig1.png)

3. I verified my setup thus far by putting the public IP address of the LoadBalancer in the browser and voila!!!
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/LB-PublicIP.png)


4. On each of the web-servers, I unmounted the logs directory from the `/mnt/logs` directory on the NFS Server by running;
```bash
sudo umount -l /var/log/httpd
```
> used the -l as it was returning a mount point busy error.

5. I restarted both servers and anytime i refreshed webpage of the loadbalancer  I was able to read their access logs (increasing in turns) by running 
```bash
sudo tail -f /var/log/httpd/access_log
```
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project12/LocalLogs.png)

NOTES: This was an interesting one as i learnt of the various Load balancing comcepts and some scenarios where they can be deployed.
