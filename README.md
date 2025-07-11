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
- Instance type- t2.micro
- Select key pair and click on launch instance
- Copy Public IP and connect to server through kitty/putty
- Select downloaded key pair to connect
- Set system hostname and timezone to identify the server.
#### $sudo hostnamectl set-hostname dev1
#### $sudo timedatectl set-timezone Asia/Kolkata
