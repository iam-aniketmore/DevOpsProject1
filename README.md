# DevOps CI/CD Pipeline with Jenkins, Maven, GitHub, Ansible, Docker
### Agenda
- Setup Jenkins
- Setup & Configure Maven and Git
- Integrate GitHub and Maven with Jenkins
- Setup Docker Host
- Integrate Docker with Jenkins using Ansible
- Build & Deploy process using Jenkins
- Test the Deployment
  
### Prerequisites
- AWS account (EC2 Ubuntu/AL2023 instances)
- GitHub account
- utty/Kitty to access the server
- Familiarity with Docker, Git, Ansible
## Step 1: Setup Developer System on AWS EC2 Instance
- Launch a EC2 Instance - Amazon linux
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/daab318d-dc4d-4168-9737-7a273ae48276" />

- Instance type- t2.micro
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/77314d55-075d-4ffa-938b-9759e1f43ab3" />

- Select key pair and click on launch instance
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/d646d6d7-393f-4bd2-b4d7-b0bf417cee0a" />

- Copy Public IP and connect to server through kitty/putty
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/42fe90d4-94fa-4176-87be-1bf688f6b3ad" />
<img width="432" height="550" alt="image" src="https://github.com/user-attachments/assets/7816a80e-8577-4716-a9dd-52d5f057427d" />

- Select downloaded key pair to connect
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/7eb99cf7-bfbd-4099-9e70-a80d12b73ed2" />

#### Set system hostname and timezone to identify the server.
* $sudo hostnamectl set-hostname dev1
* $sudo timedatectl set-timezone Asia/Kolkata
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/265f705f-966c-4b7c-b0db-e7f566205fa6" />

  
#### Install Git
* #yum install git -y
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/357a8b75-ab49-46de-bd9f-c563814e6906" />


#### Create project directory and initialize Git

* #mkdir /devpro1   (creates a workspace)
* #cd /devpro1
* #git init		(Initiate as git repo)
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/a2473b98-c9fa-4be1-a56f-3d309a7f20ed" />

  
#### Create user and email for git operation
* #git config --global user.name aniket
* #git config --global user.email aniketmore@gmail.com

#### These are used to configure your Git identity — specifically, who is making commits.

<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/bd10c1af-304f-4493-9ced-6d5d9385a54a" />

#### Stage all files for commit
* #git add . 

#### Commit the staged files with a message
* #git commit -m ‘first commit’ .   

#### Check the current status of the repository
* #git status
  
<img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/4a352034-9900-4998-abd3-1e9d06c6b11a" />

## Step2: Create repository in github.com ( DevOpsProject1)

### Open browser - github.com -create repository - "DevOpsProject1"

#### Open terminal on dev system - 
#### Set the default branch to main (instead of master)
* #git branch -M main 

#### Link the local repository with your GitHub remote repo
* #git remote add origin https://github.com/iam-aniketmore/DevOpsProject1.git
  
#### Push your local commits to GitHub
* #git push https://ghp_8Efyyta8V6l7fjdjdfdhhdfhhawtpZb3e@github.com/iam-aniketmore/DevOpsProject1.git     (Push code to github)

<img width="604" height="117" alt="image" src="https://github.com/user-attachments/assets/23e1db81-2bd5-4aaf-9168-ea9cf9b22f1a" />

Goto github repository and refresh page - verify code is uploaded.

<img width="621" height="260" alt="image" src="https://github.com/user-attachments/assets/76aba7ac-59e2-4385-afd6-7ef56f547e6a" />

## Step3: Setup jenkins server ( use AL2023 t3.small )

- Select Amazon linux - Select t3 small - select key pair and launch instance.

<img width="627" height="378" alt="image" src="https://github.com/user-attachments/assets/d3375ced-abd9-4e35-8354-f29566b35055" />

### 1.Login on Jenkins_Server:

#### Set system hostname and timezone to identify the server.

* $sudo hostnamectl set-hostname jenkins
* $sudo timedatectl set-timezone Asia/Kolkata

### 2.Install java

* #yum update -y
* #yum install java-11* -y
* #java -version

### 3.Create tomcat user and group

* #groupadd --system tomcat
* #useradd -d /usr/share/tomcat -r -s /bin/false -g tomcat tomcat

#### Why this step-
#### This create a dedicated system user and group for Tomcat so that it doesn’t run as root.

### 4.Install Tomcat 9 on Amazon Linux 2

* #wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.63/bin/apache-tomcat-9.0.63.tar.gz

<img width="636" height="68" alt="image" src="https://github.com/user-attachments/assets/df079120-573a-4f91-a20b-9dcd38caae64" />

### 5.Use tar command line tool to extract downloaded archive.

* #tar -xvzf apache-tomcat-9.0.63.tar.gz -C /usr/share/

### 6.Create Symlink to the folder /usr/share/tomcat. This is for easy updates.

* #ln -s /usr/share/apache-tomcat-9.0.63/ /usr/share/tomcat

#### Why this step-
#### This make version upgrades easier in the future

### 7.Update folder permissions

* #chown -R tomcat:tomcat /usr/share/tomcat
* #chown -R tomcat:tomcat /usr/share/apache-tomcat-9.0.63/

##### Why this step-
##### These commands change the ownership of Tomcat’s directories to the tomcat user and group we created earlier. It ensures that only the Tomcat user can read/write/execute within its directories, which enhances security and proper functioning.
##### -R means recursive – it applies to all subdirectories and files.

### 8.Create Tomcat Systemd service

##### Run following code to create tomcat service in a single command on terminal
* #tee /etc/systemd/system/tomcat.service<<EOF
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

##### Why this step-
##### This will create a systemd service unit file to manage Tomcat like any other Linux service. This will allow us to easily start, stop, enable, or check the status of Tomcat using systemctl commands.


### 8.Enable and start tomcat service:

* #systemctl daemon-reload
* #systemctl start tomcat
* #systemctl enable tomcat
* #systemctl status tomcat

<img width="642" height="158" alt="image" src="https://github.com/user-attachments/assets/20525bd9-09c9-4b11-a997-adb28d049fe8" />

### 9.Configure Tomcat Authentication

##### Why this step-
##### We have to edit Tomcat configuration file to enable Admin and Manager UI roles.
##### Tomcat comes without access to its web GUI (admin & manager) by default for security reasons. So, to use the web dashboard, you must define roles and users manually in the tomcat-users.xml file.

* #vim /usr/share/tomcat/conf/tomcat-users.xml
#### Add below lines before closing with </tomcat-users>

  <role rolename="admin-gui"/>
  <role rolename="manager-gui"/>
  <user username="admin" password="111" fullName="Administrator" roles="admin-gui,manager-gui"/>
  :wq

<img width="626" height="209" alt="image" src="https://github.com/user-attachments/assets/b919b405-9564-4a80-893c-3424f6a60e08" />

### 10.Configure Apache web server as a proxy for Tomcat server. First install httpd package.

##### Why this step-
##### Installing Apache (httpd) allows you to forward traffic from port 80 to Tomcat on 8080, improving accessibility, flexibility, and security.

* #yum install httpd  -y

### 12.Create VirtualHost file for Tomcat Admin web interface:

##### Why this step-
##### We are telling the Apache server how to forward web traffic to your Tomcat server (running on port 8080 or 8009) using proxy directives.

* #vim /etc/httpd/conf.d/tomcat_manager.conf
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

:wq

### 13.Restart and enable httpd service:

* #systemctl start httpd
* #systemctl enable httpd
* #systemctl status httpd

### 14.Open Web Browser and open address

##### http://192.168.1.111
##### login tomcat with proper authentication

##### Manage App
##### username: admin	
##### password: 111

<img width="516" height="361" alt="image" src="https://github.com/user-attachments/assets/85c5b959-76b5-43c3-9dae-c8dc22469097" />
<img width="438" height="281" alt="image" src="https://github.com/user-attachments/assets/07c7bfb7-d49e-4bc9-b28e-a0203afb6ab7" />

### 5. Open terminal and configure jenkins

* #cd /usr/share/tomcat/webapps
* #ls

#### 16.	Download jenkins war file
* #wget wget https://updates.jenkins.io/download/war/2.462/jenkins.war

#### Restart tomcat service and open jenkins
* #systemctl restart tomcat

### 17.	Open Web Browser and open address
#### http://13.214.176.70/jenkins


### 18.Login into jenkins

#### Go to linux get default password using following command

* #cat /usr/share/tomcat/.jenkins/secrets/initialAdminPassword

<img width="549" height="120" alt="image" src="https://github.com/user-attachments/assets/643a20a4-e2ca-4cd9-a1c0-b1b12e90b389" />

#### Getting Started - Customize Jenkins – Install suggested plugins.

<img width="562" height="289" alt="image" src="https://github.com/user-attachments/assets/9f6a581c-332d-4b54-9acc-1f87bb5fe76a" />

### 19.Set jenkins new credentials
#### username: admin
#### password: 111

<img width="492" height="338" alt="image" src="https://github.com/user-attachments/assets/83a20308-79f5-45fd-bb50-0eee2704226b" />

#### Login in Jenkin Dashboard. 

<img width="543" height="242" alt="image" src="https://github.com/user-attachments/assets/c0e53fb4-ea94-44c8-b509-07a0235b3248" />



