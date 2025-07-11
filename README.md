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
- <img width="529" height="373" alt="image" src="https://github.com/user-attachments/assets/daab318d-dc4d-4168-9737-7a273ae48276" />

- Instance type- t2.micro
- Select key pair and click on launch instance
- Copy Public IP and connect to server through kitty/putty
- Select downloaded key pair to connect
- Set system hostname and timezone to identify the server.
* $sudo hostnamectl set-hostname dev1
* $sudo timedatectl set-timezone Asia/Kolkata
  
#### Install Git
* #yum install git -y 

#### Create project directory and initialize Git

* #mkdir /devpro1   (creates a workspace)
* #cd /devpro1
* #git init		(Initiate as git repo)
  
#### Create user and email for git operation
* #git config --global user.name aniket
* #git config --global user.email aniketmore@gmail.com

#### These are used to configure your Git identity — specifically, who is making commits.
#### Stage all files for commit
* #git add . 

#### Commit the staged files with a message
* #git commit -m ‘first commit’ .   

#### Check the current status of the repository
* #git status

