# DevOps CI/CD Pipeline with Jenkins, Maven, GitHub, Ansible, Docker
### Agenda
- Setup Jenkins
  ___________________________________________________________________________
- Setup & Configure Maven and Git
- Integrate GitHub and Maven with Jenkins
- Setup Docker Host
- Integrate Docker with Jenkins using Ansible
- Build & Deploy process using Jenkins
- Test the Deployment
___________________________________________________________________________
### Prerequisites
- AWS account (EC2 Ubuntu/AL2023 instances)
- GitHub account
- utty/Kitty to access the server
- Familiarity with Docker, Git, Ansible

  -----------------------------------------------------------------------------------------------

## Step 1: Setup Developer System on AWS EC2 Instance
- Launch a EC2 Instance - Amazon linux
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/daab318d-dc4d-4168-9737-7a273ae48276" />

- Instance type- t2.micro
  ___________________________________________________________________________
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/77314d55-075d-4ffa-938b-9759e1f43ab3" />

- Select key pair and click on launch instance
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/d646d6d7-393f-4bd2-b4d7-b0bf417cee0a" />

- Copy Public IP and connect to server through kitty/putty
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/42fe90d4-94fa-4176-87be-1bf688f6b3ad" />
<img width="432" height="550" alt="image" src="https://github.com/user-attachments/assets/7816a80e-8577-4716-a9dd-52d5f057427d" />

- Select downloaded key pair to connect
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/7eb99cf7-bfbd-4099-9e70-a80d12b73ed2" />

#### Set system hostname and timezone to identify the server.
```
$sudo hostnamectl set-hostname dev1
$sudo timedatectl set-timezone Asia/Kolkata
```
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/265f705f-966c-4b7c-b0db-e7f566205fa6" />

  
#### Install Git
```
#yum install git -y
  ```
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/357a8b75-ab49-46de-bd9f-c563814e6906" />


#### Create project directory and initialize Git
```
#mkdir /devpro1   (creates a workspace)
#cd /devpro1
#git init		(Initiate as git repo)
 ``` 
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/a2473b98-c9fa-4be1-a56f-3d309a7f20ed" />

  
#### Create user and email for git operation
```
#git config --global user.name aniket
#git config --global user.email aniketmore@gmail.com
```
#### These are used to configure your Git identity — specifically, who is making commits.

<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/bd10c1af-304f-4493-9ced-6d5d9385a54a" />

#### Stage all files for commit
```
#git add . 
```
#### Commit the staged files with a message
```
#git commit -m ‘first commit’ .   
```
#### Check the current status of the repository
```
#git status
  ```
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/4a352034-9900-4998-abd3-1e9d06c6b11a" />

-----------------------------------------------------------------------------------------------

## Step2: Create repository in github.com ( DevOpsProject1)

### Open browser - github.com -create repository - "DevOpsProject1"

#### Open terminal on dev system - 
#### Set the default branch to main (instead of master)
```
#git branch -M main 
```
#### Link the local repository with your GitHub remote repo
```
#git remote add origin https://github.com/iam-aniketmore/DevOpsProject1.git
 ``` 
#### Push your local commits to GitHub
```
#git push https://ghp_8Efyyta8V6l7fjdjdfdhhdfhhawtpZb3e@github.com/iam-aniketmore/DevOpsProject1.git     (Push code to github)
```
<img width="604" height="117" alt="image" src="https://github.com/user-attachments/assets/23e1db81-2bd5-4aaf-9168-ea9cf9b22f1a" />

Goto github repository and refresh page - verify code is uploaded.

<img width="621" height="260" alt="image" src="https://github.com/user-attachments/assets/76aba7ac-59e2-4385-afd6-7ef56f547e6a" />

-----------------------------------------------------------------------------------------------

## Step3: Setup jenkins server ( use AL2023 t3.small )

- Select Amazon linux - Select t3 small - select key pair and launch instance.

<img width="627" height="378" alt="image" src="https://github.com/user-attachments/assets/d3375ced-abd9-4e35-8354-f29566b35055" />

### 1.Login on Jenkins_Server:

#### Set system hostname and timezone to identify the server.
```
$sudo hostnamectl set-hostname jenkins
$sudo timedatectl set-timezone Asia/Kolkata
```
### 2.Install java
```
#yum update -y
#yum install java-11* -y
#java -version
```
### 3.Create tomcat user and group
```
#groupadd --system tomcat
#useradd -d /usr/share/tomcat -r -s /bin/false -g tomcat tomcat
```
#### Why this step-
#### This create a dedicated system user and group for Tomcat so that it doesn’t run as root.

### 4.Install Tomcat 9 on Amazon Linux 2
```
#wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.63/bin/apache-tomcat-9.0.63.tar.gz
```
<img width="636" height="68" alt="image" src="https://github.com/user-attachments/assets/df079120-573a-4f91-a20b-9dcd38caae64" />

### 5.Use tar command line tool to extract downloaded archive.
```
#tar -xvzf apache-tomcat-9.0.63.tar.gz -C /usr/share/
```
### 6.Create Symlink to the folder /usr/share/tomcat. This is for easy updates.
```
#ln -s /usr/share/apache-tomcat-9.0.63/ /usr/share/tomcat
```
#### Why this step-
#### This make version upgrades easier in the future

### 7.Update folder permissions
```
#chown -R tomcat:tomcat /usr/share/tomcat
#chown -R tomcat:tomcat /usr/share/apache-tomcat-9.0.63/
```
##### Why this step-
##### These commands change the ownership of Tomcat’s directories to the tomcat user and group we created earlier. It ensures that only the Tomcat user can read/write/execute within its directories, which enhances security and proper functioning.
##### -R means recursive – it applies to all subdirectories and files.

### 8.Create Tomcat Systemd service

##### Run following code to create tomcat service in a single command on terminal
```
#tee /etc/systemd/system/tomcat.service<<EOF
[Unit]
Description=Tomcat Server
After=syslog.targetnetwork.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment='JAVA_OPTS=-Djava.awt.headless=true'
Environment=CATALINA_HOME=/usr/share/tomcat
Environment=CATALINA_BASE=/usr/share/tomcat
Environment=CATALINA_PID=/usr/share/tomcat/temp/tomcat.pid
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M'
ExecStart=/usr/share/tomcat/bin/catalina.sh start
ExecStop=/usr/share/tomcat/bin/catalina.sh stop


[Install]
WantedBy=multi-user.target
EOF
```
##### Why this step-
##### This will create a systemd service unit file to manage Tomcat like any other Linux service. This will allow us to easily start, stop, enable, or check the status of Tomcat using systemctl commands.


### 8.Enable and start tomcat service:
```
#systemctl daemon-reload
#systemctl start tomcat
#systemctl enable tomcat
#systemctl status tomcat
```
<img width="642" height="158" alt="image" src="https://github.com/user-attachments/assets/20525bd9-09c9-4b11-a997-adb28d049fe8" />

### 9.Configure Tomcat Authentication

##### Why this step-
##### We have to edit Tomcat configuration file to enable Admin and Manager UI roles.
##### Tomcat comes without access to its web GUI (admin & manager) by default for security reasons. So, to use the web dashboard, you must define roles and users manually in the tomcat-users.xml file.
```
#vim /usr/share/tomcat/conf/tomcat-users.xml
```
#### Add below lines before closing with </tomcat-users>
```
  <role rolename="admin-gui"/>
  <role rolename="manager-gui"/>
  <user username="admin" password="111" fullName="Administrator" roles="admin-gui,manager-gui"/>
  :wq
```
<img width="626" height="209" alt="image" src="https://github.com/user-attachments/assets/b919b405-9564-4a80-893c-3424f6a60e08" />

### 10.Configure Apache web server as a proxy for Tomcat server. First install httpd package.

##### Why this step-
##### Installing Apache (httpd) allows you to forward traffic from port 80 to Tomcat on 8080, improving accessibility, flexibility, and security.
```
#yum install httpd  -y
```
### 12.Create VirtualHost file for Tomcat Admin web interface:

##### Why this step-
##### We are telling the Apache server how to forward web traffic to your Tomcat server (running on port 8080 or 8009) using proxy directives.
```
#vim /etc/httpd/conf.d/tomcat_manager.conf
<VirtualHost *:80>
ServerAdmin root@localhost
ServerName tomcat.example.com
DefaultType text/html
ProxyRequests off
ProxyPreserveHost On
ProxyPass / http://localhost:8080/
ProxyPassReverse / http://localhost:8080/
</VirtualHost>

<VirtualHost *:80>
ServerName ajp.example.com
ProxyRequests Off
ProxyPass / ajp://localhost:8009/
ProxyPassReverse / ajp://localhost:8009/
</VirtualHost>
```

### 13.Restart and enable httpd service:
```
#systemctl start httpd
#systemctl enable httpd
#systemctl status httpd
```
### 14.Open Web Browser and open address

##### http://192.168.1.111
##### login tomcat with proper authentication

##### Manage App
##### username: admin	
##### password: 111

<img width="516" height="361" alt="image" src="https://github.com/user-attachments/assets/85c5b959-76b5-43c3-9dae-c8dc22469097" />
<img width="438" height="281" alt="image" src="https://github.com/user-attachments/assets/07c7bfb7-d49e-4bc9-b28e-a0203afb6ab7" />

### 5. Open terminal and configure jenkins
```
#cd /usr/share/tomcat/webapps
#ls
```
### 16.	Download jenkins war file
```
#wget wget https://updates.jenkins.io/download/war/2.462/jenkins.war
```
#### Restart tomcat service and open jenkins
```
#systemctl restart tomcat
```
### 17.	Open Web Browser and open address
#### http://13.214.176.70/jenkins


### 18.Login into jenkins

#### Go to linux get default password using following command
```bash
#cat /usr/share/tomcat/.jenkins/secrets/initialAdminPassword
```
<img width="549" height="120" alt="image" src="https://github.com/user-attachments/assets/643a20a4-e2ca-4cd9-a1c0-b1b12e90b389" />

#### Getting Started - Customize Jenkins – Install suggested plugins.

<img width="562" height="289" alt="image" src="https://github.com/user-attachments/assets/9f6a581c-332d-4b54-9acc-1f87bb5fe76a" />

### 19.Set jenkins new credentials
#### username: admin
#### password: 111

<img width="492" height="338" alt="image" src="https://github.com/user-attachments/assets/83a20308-79f5-45fd-bb50-0eee2704226b" />

#### Login in Jenkin Dashboard. 

<img width="543" height="242" alt="image" src="https://github.com/user-attachments/assets/c0e53fb4-ea94-44c8-b509-07a0235b3248" />

-----------------------------------------------------------------------------------------------

### Step4:  Setup required tools path such as java,maven etc.

##### Why this step-
##### Configured Java environment path by updating .bash_profile with JAVA_HOME and modifying PATH. This ensures Java is globally accessible for all build tools like Maven and Jenkins.

#### 1.Set java path

#####- Verify Version
```
#java -version
```

##### Update path in bash profile
```
#cd
#vim .bash_profile
```
```
JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64/bin/java
PATH=$PATH:$JAVA_HOME:$HOME/bin
```
```
#echo $PATH
#source .bash_profile
#echo $PATH
```
### 2.Install Maven Amazon Linux AMI2

##### Following are the set of commands need to be executed sequentially to install maven.
```
#cd /opt/
#wget  https://dlcdn.apache.org/maven/maven-3/3.8.8/binaries/apache-maven-3.8.8-bin.tar.gz
#tar -xvzf  apache-maven-3.8.8-bin.tar.gz
#rm -rvf  *.gz
#mvn –version
```
### 3.Set path for maven and java (This ensures both tools are available system-wide for Jenkins and terminal operations.) 
```
#cd
#vim .bash_profile
JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64/bin/java
MAVEN_HOME=/opt/apache-maven-3.8.8
M2=/opt/apache-maven-3.8.8/bin/
PATH=$PATH:$JAVA_HOME:$MAVEN_HOME:$M2:$HOME/bin

#source .bash_profile
#echo $M2
#echo $PATH
#echo $MAVEN_HOME
```

<img width="603" height="263" alt="image" src="https://github.com/user-attachments/assets/89f87956-a9c1-4f64-8c90-b2a6295bad2d" />

### 4.Install git:
```
#yum update -y
#yum install git -y
#git  --version
#which git
```
-----------------------------------------------------------------------------------------------

### Step 5:  Create project for pull code and build war file

##### 1.Open Jenkins

##### http://localhost/jenkins

#### Dashboard – Manage Jenkins – Tools – git – Name (default) – path to git execute (/bin/git)

#### Maven – add maven – Name (mvn) – MAVEN_HOME(/opt/apache-maven-3.8.8/)– 
#### Apply – Save.

##### Why this step - 
##### Jenkins needs to know where Git and Maven are installed on your system to use them in build pipelines. Without this configuration, Jenkins jobs will fail when they try to use git or mvn.

<img width="528" height="310" alt="image" src="https://github.com/user-attachments/assets/970ca14a-ac56-4cbd-9626-23db100987c2" />

<img width="555" height="422" alt="image" src="https://github.com/user-attachments/assets/20fb554f-0c15-4138-b948-8a2777ff210f" />

### 2.Open Jenkins.

#### http://localhost/jenkins

#### Dashboard – New Item – Enter new item name (devpro2025) – freestyle project – ok – source code management – git – repository url ( https://github.com/iam-aniketmore/DevOpsProject1.git ) 
#### Build Steps (invoke top-level maven targets ) – Maven version (mvn) – Goals ( clean install package ) – Apply – Save.
Also set branch main

##### Why this step- 
##### We are now creating a Jenkins job that will:
##### - Pull the source code from GitHub
##### - Use Maven to build it
##### - Run steps like clean install package
##### - This enables automated CI/CD

<img width="572" height="353" alt="image" src="https://github.com/user-attachments/assets/c1619866-407d-4905-bd96-9412fae569b2" />

<img width="554" height="289" alt="image" src="https://github.com/user-attachments/assets/5a67c66c-7f73-4a80-8b23-c6a4288b0052" />

<img width="554" height="287" alt="image" src="https://github.com/user-attachments/assets/b924a0ee-1342-4d9c-945e-f215b5239761" />

<img width="521" height="417" alt="image" src="https://github.com/user-attachments/assets/d26465e9-e5c6-45fc-a3a1-8d045ce65121" />

#### Dashboard – devpro2025 – Build Now – console output.

<img width="573" height="394" alt="image" src="https://github.com/user-attachments/assets/da7332eb-9362-4f79-b729-a218d72e1ee0" />

-----------------------------------------------------------------------------------------------

## Step 6. Create ansible server: ( AL 2023)
-------------------------------------------------------------------------------------
### 1.use following bootstrap code for launch ec2 for ansible
```
#!/bin/bash
yum update -y
yum install ansible* -y
hostnamectl  set-hostname ansible
useradd itadmin
echo 111 | passwd --stdin itadmin
echo 111 | passwd --stdin root
echo "itadmin  ALL=(ALL)   NOPASSWD: ALL" >> /etc/sudoers
sed 's/PasswordAuthentication no/PasswordAuthentication yes/' -i /etc/ssh/sshd_config
echo PermitRootLogin yes >> /etc/ssh/sshd_config
systemctl restart sshd
```
-----------------------------------------------------------------------------------------------

## Step 7. Create Docker server: ( AL 2023)

### 1. use following bootstrap code for launch ec2 for docker-node1
```
#!/bin/bash
yum update -y
hostnamectl  set-hostname docker-node1
useradd itadmin
echo 111 | passwd --stdin itadmin
echo 111 | passwd --stdin root
echo "itadmin  ALL=(ALL)   NOPASSWD: ALL" >> /etc/sudoers
sed 's/PasswordAuthentication no/PasswordAuthentication yes/' -i /etc/ssh/sshd_config
echo PermitRootLogin yes >> /etc/ssh/sshd_config
systemctl restart sshd
```
-----------------------------------------------------------------------------------------------

## Step 8: Create connectivity between ansible and docker-nodes

#### Login as itadmin and establish ssh based authentication with Docker_server
```
$su itadmin
$ssh-keygen
```

<img width="592" height="362" alt="image" src="https://github.com/user-attachments/assets/fd6721ae-9347-4007-9844-1471a90774d9" />

### 1.Update /etc/hosts file
```
$sudo  vim /etc/hosts
172.31.37.154 docker-node1
```
##### Why this step-
##### To make it easier to connect to remote nodes by name (instead of IP), we add their IP–hostname mappings to the /etc/hosts file.

<img width="616" height="173" alt="image" src="https://github.com/user-attachments/assets/a0a877a1-59d5-4a44-b72b-2496110a42d0" />

### 2. Transfer generate ssh key to all nodes
```
$ssh-copy-id  itadmin@docker-node1
```

##### Why this step-
##### This allows Ansible to connect and execute commands on remote hosts without prompting for a password every time.

-----------------------------------------------------------------------------------------------

### 3. Update inventory file
```
$sudo mkdir /opt/project
$sudo  vim  /opt/project/inventory
[docker]
docker-node1
docker-node2
```

##### This file tells Ansible which servers (nodes) it should manage.

-----------------------------------------------------------------------------------------------
### 3.Create ansible.cfg file
```
$sudo vim /opt/project/ansible.cfg
[defaults]
inventory=/opt/project/inventory
remote_user=itadmin
host_key_checking=false
[privilege_escalation]
become=true
become_user=root
become_method=sudo
become_ask_pass=false
```
##### Why this step-
##### This ensures Ansible knows where to look for the inventory file and how to handle privilege escalation and remote access.
-----------------------------------------------------------------------------------------------

### 5. Check ansible client (docker server) is reachable through ansible server
```
$cd   /opt/project/
$ansible  all  -m  ping
```
-----------------------------------------------------------------------------------------------

### 6. Create directory for store project webapp
```
$sudo  mkdir   /opt/docker
$sudo chown -R itadmin:itadmin /opt/docker
$sudo chown -R itadmin:itadmin /opt/project
```
##### Why this step-
##### To manage and deploy the web application code effectively on Docker nodes, we first need to create a dedicated directory with appropriate permissions.
-----------------------------------------------------------------------------------------------

### 7.Create playbook for manage containers on docker-nodes
##### Why this step-
##### We’ll now create an Ansible playbook that automates the full container lifecycle on all Docker nodes. This includes installing Docker, managing images/containers, copying files, and deploying the WAR file in a Tomcat container inside Docker.
```
$vim /opt/project/create-docker-container.yml
---
 - name: create docker container
   hosts: all
   ignore_errors: yes
   tasks:
     - name: install docker
       yum:
         name: docker
         state: present
     - name: start and enable docker
       service:
         name: docker
         state: started
         enabled: yes
     - name: add user into group
       command: usermod -aG docker itadmin
     - name: create docker directory
       file:
         path: /opt/docker
         state: directory
     - name: change ownership
       command: chown -R itadmin:itadmin /opt/docker
     - name: stop the running container
       shell: docker stop webapp
     - name: remove container
       shell: docker rm -f webapp
     - name: remove container image
       shell: docker rmi -f myapp:latest
     - name: transfer docker file
       copy:
         src: /opt/docker/Dockerfile
         dest: /opt/docker/
     - name: trasfer web appplication
       copy:
         src: /opt/docker/webapp.war
         dest: /opt/docker/
     - name: build docker image
       shell: cd /opt/docker; docker build -t myapp.
     - name: create docker tomcat container
       shell: docker run -dt --name webapp -p 80:8080 myapp:latest

```

-----------------------------------------------------------------------------------------------

## Step 9: Update in jenkins project

### 1.Login Jenkins & install required plugins
##### Why this step-
##### To enable Jenkins to communicate with remote servers (like Ansible control node) and deploy applications using SSH, we need to install the “Publish Over SSH” plugin.

#### Dashboard- manage Jenkins – manage plugins – available – install “publish over ssh” plugins – restart Jenkins.

<img width="638" height="180" alt="image" src="https://github.com/user-attachments/assets/8f995003-aec8-4dc5-a071-3a1f9412c2f8" />

### 2. Add ansible credentials 
##### Why this step-
##### Add the Ansible Control Node (host) credentials to Jenkins. This will allow Jenkins to send files and trigger commands (like Ansible playbooks) on the remote server.

#### Dashboard – manage Jenkins – configure system – system –SSH Servers (add ) – Name (Ansible_Server) Hostname (192.168.1.112) – username (itadmin) – Advanced - use password authentication or use a different key (111) – Test Configuration – Apply - Save.

<img width="498" height="402" alt="image" src="https://github.com/user-attachments/assets/0c599f26-6ecf-4b43-8cda-0749e39766fc" />

### 3. Update devpro2025
##### Why this step-
##### After Jenkins builds our project and generates the .war file (web application archive), we need to transfer that .war file to the Ansible server, where it will be deployed using Docker.

#### Open Dashboad – devpro2024 – Configure-
#### Now click on “Post Build Action” – send build artifacts over SSH – Name (Ansible_Server) – Source files (webapp/target/*.war) – Remove prefix (webapp/target/) – Remote directory (//opt/docker)

##### After the .war file is sent, we also need the Dockerfile on the Ansible server, because it defines how the Docker image will be built..

#### Add Server
#### Name (Ansible_Server) – Source files (Dockerfile) – Remote directory (//opt/docker)

#####trigger the deployment using Ansible.

#### Add Server
#### Exec command
#### cd /opt/project; ansible-playbook create-docker-container.yml

##### Save - Apply 

<img width="484" height="298" alt="image" src="https://github.com/user-attachments/assets/3c39d6be-8569-4aa4-85a7-56190b796f62" />

<img width="475" height="361" alt="image" src="https://github.com/user-attachments/assets/846fb2bf-75ae-4f07-932b-283dbae2ef25" />

### 4.Check and verify on browser
##### You will get this error

<img width="752" height="195" alt="image" src="https://github.com/user-attachments/assets/bba1b8e8-3ce6-430a-9767-41527f9d4ccd" />

##### From the above screenshot, you can see that although there is a 404 error, it also displays Apache Tomcat at the bottom which means the installation is successful however this is a known issue with the Tomcat docker image that we will fix in the next steps.

##### The above issue occurs because whenever we try to access the Tomcat server from the browser it will look for the files in /webapps directory which is empty and the actual files are being stored in /webapps.dist.
##### So in order to fix this issue we will copy all the content from webapps.dist to webapps directory and that will resolve the issue.

-----------------------------------------------------------------------------------------------

## Step10: For Verify

### 1.Open docker docker-node1 terminal and check docker info
```
#docker images
#docker ps
#docker ps -a
#ifconfig
```
### 2. Open web browser –
#### http://dockerip/webapp
#### You won’t see any details

### 3. Open jenkins and Run Project
#### Go to Jenkins dashboard – run created project (right click and build)

#### Now again repeat step1 to verify docker info

### 4. Open web browser –
#### http://dockerip/webapp

<img width="618" height="236" alt="image" src="https://github.com/user-attachments/assets/a373381a-290d-4393-8f84-0305a147515a" />
