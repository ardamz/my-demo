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

4. I configured `Jenkins` by perfor,ming the following steps;
    *  I created a new Freestyle project called `ansible` in Jenkins and pointed it to my `ansible-config-mgt` repository.
    -  I configured Webhook in GitHub and set webhook to trigger `ansible` build.
    *  I configured a Post-build job to save all (**) files.
    
5. I tested my setup by making some changes to the main branch and made sure that builds starts automatically and Jenkins saved the files (build artifacts) as shown below;

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/Artifacts.png)


## STEP 2.  **Prepare the Development Environment using Visual Studio Code (VSC)**

1. VSC was configured to connect to my `ansible-config-mgt` repository.

2. i then cloned the `ansible-config-mgt` repository to my Jenkins-AAnsible Instance.
```bash
git clone https://github.com/ardamz/ansible-config-mgt.git
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/GitClone.png)

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

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/Branch.png)

2. I created `playbooks` (used to store all your playbook files) and `inventotory` (used to keep your hosts organised) directories.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/Directory.png)

3. Created a `common.yml` file in the `playbooks` directory.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/Common.png)

4. Within the inventory directory,I created an inventory file (.yml) for each environment (Development, Staging, Testing and Production) dev, staging, uat, and prod respectively.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/Inventories.png)


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
![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/SshAdd.png)

3. Access the `Jenkins-Ansible` server using ssh-agent by running

```bash
 ssh -A ubuntu@3.80.32.204
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/SshAgentLogin.png)

4. After setting up and verifying the ssh-agent, I updated the `inventory/dev.yml` file with the details of the various servers.


## STEP 5.  **Create A Common Playbook**

1. I populated the playbooks/common.yml with the following lines of code

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

## STEP 6 - **Update GIT with the latest code**

After editing the `dev.yml` and `common.yml` files, I pushed them to the git repo by running the following lines;

```bash
# check the status of the git repository
git status

# Add all untracked file(s) to the repo to monitor changes
git add .

# commit all saved changes to the repo
git commit -m "commit message"
```
Acting as a reviewer, on the website I reviewed the changes to the codes, found them acceptable and merged them to the main branch.
on the terminal, i checked out from the `Prj_11` branch into the main branch, and pulled down the latest changes to the machine


## Step 7 - **Run first Ansible test**
I navigated in the `ansible-config-mgt` containing the latest changes to the code and ran the following lines of codes;

```bash
ansible-playbook -i inventory/dev.yml playbooks/common.yml
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/1play1.png)
> Play 1 input.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/Play1.png)
> Play 1 output.

Going through the various servers, i was able to verify the intallation of wireshark

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/DBWireshark.png)


## Step 8 - **Run second Ansible test**

I created a new file called `common2.yml` and populaed it as follow;

```bash
---
- name: create directory, file and set timezone on all servers
  hosts: webservers, nfs, db, lb
  become: yes
  tasks:
    - name: create a directory
      file:
        path: /home/sample
        state: directory

    - name: create a file
      file:
        path: /home/sample/ansible.txt
        state: touch

    - name: set timezone
      timezone:
        name: Africa/Lagos
```
> This simple playbook creates a `ansible.txt` file in a `sample` directory and changes the timezones on all the server groups.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/project11/Verified.png)
> Play 2 verified on the NFS & LB.

