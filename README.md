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
  <img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/695c52a5-5c74-4794-b2d9-1a30e9267563" />

- Select downloaded key pair to connect
  <img width="650" height="280" alt="image" src="https://github.com/user-attachments/assets/7eb99cf7-bfbd-4099-9e70-a80d12b73ed2" />

- Set system hostname and timezone to identify the server.
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


