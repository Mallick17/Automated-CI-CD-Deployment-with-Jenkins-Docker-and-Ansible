# Automated Deployment with Jenkins, Docker and Ansible
This documentation explains the steps followed for deploying a project using Docker, Ansible, and Jenkins, including the configuration of tools and execution of a pipeline for a complete automation workflow.
### Overview
This project demonstrates the deployment of a finance.war web application using Jenkins, Docker, Tomcat, and Ansible. The application is built and packaged in Jenkins, deployed using Docker, and hosted on a Tomcat server. The deployed application is accessible via a public IP and port.
# 1. Prerequisites
## Installation on Root User
- 1. **Install Java 17**:
  ```bash
  yum install java-17* -y
  ```
- 2. **Install Git**:
  ```bash
  yum install git -y
  ```
- 3. **Setup Docker & Docker Compose**
  - Follow standard Installation
  [Install Docker & Docker Compose](https://github.com/Mallick17/Docker.git)
- 4. **Set Up Ansible**
  - Install and configure Ansible on the Master node to manage the Worker node.Follow standard Installation
  [Install Ansible & Configure](https://github.com/Mallick17/Ansible.git)

# 2. Jenkins Setup
## Installing Jenkins
  - To install Jenkins on a Red Hat-based system, prefer jenkins installation guide for Red Hat [Jenkins Installation page](https://www.jenkins.io/doc/book/installing/linux/#long-term-support-release-3)
  - Enable and Start Jenkins<br>
    Enable Jenkins to start on boot, and start the Jenkins service immediately.
  ```bash
  systemctl enable jenkins
  systemctl start jenkins
  ```
  - Verify Jenkins Installation<br>
    Jenkins typically runs on port 8080. Ensure it is accessible by checking the connection at `http://<your-server-ip>:8080`.
  - Retrieve Jenkins Admin Password<br>
    The initial admin password for Jenkins is stored in the following location. 
    Retrieve it using the command below:
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
## Configure Jenkins
### 1. Install Maven Plugin in Jenkins
To build Java projects with Maven, you must install the Maven plugin in Jenkins.
- Log in to Jenkins.
- Navigate to **Manage Jenkins** → **Tools** → **Maven Installation**.
- Click **Add Maven**, provide a name (e.g., `Name_Maven`), and **Save & Apply**.

### 2. Install Plugins
- Go to **Dashboard → Manage Jenkins → Plugins → Available Plugins**.
- Search for `Publish Over SSH` and **install** it.

### 3. Configure Publish Over SSH
- Navigate to **Dashboard → Manage Jenkins → Publish Over SSH**.
- Set the following configurations
  - **Passphrase**: `1234` (password for the *ansible* user).
  - Add an **SSH Server**
    - **Name**: `ansible`
    - **Hostname**: `<Private IP of Jenkins server>`
    - **Username**: `ansible`
    - **Check** →O *Use password authentication or use a different key.*
    - **Enter passphrase/password**: `1234`
  - Click `Test Configuration` and ensure it is successful.
- **Apply and save** the configuration.

## Pipeline Configuration
1. **Create a Freestyle Job**
- On the **Jenkins dashboard**, click **New Item**.
- Enter the name `dev_deploy`, choose **Freestyle project**, and click **OK**.
2. **Configure the Job**
- In General:
  - Under **Source Code Management**, select **Git** and *provide the repository URL*.
  - Click **Apply & Save**.
  - Click **Build Now**
3. **Build Steps**
- Under **Build Steps**, click **Add build step → Invoke top-level Maven targets**.
  - Set **Maven Version** to the version installed (e.g., `Maven`).
  - Set **Goals** to `clean package`.
  - Click **Apply & Save**.
  - *Trigger the build to generate the WAR file.*
    
## Deploy Application
### 1. Prepare Dockerfile
- Log in as the `ansible` user and create a Dockerfile:
  ```bash
  su - ansible
  vi Dockerfile
  ```
  Contents of `Dockerfile`:
  ```dockerfile
  FROM tomcat:9-jre9
  MAINTAINER "gyanaranjanmallick444@gmail.com"
  COPY ./finance.war /usr/local/tomcat/webapps
  ```
### 2. Configure Additional Build Steps in Jenkins



