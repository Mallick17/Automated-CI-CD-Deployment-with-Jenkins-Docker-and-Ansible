# Complete Documentation: Jenkins Pipeline for Automated Docker Deployment with Ansible
This documentation provides a detailed step-by-step guide to set up a Jenkins pipeline for automating Docker image building, pushing to Docker Hub, and deploying a web application using Ansible.

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
  - Login Docker in the Linux Terminal
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
## 3. Configure Jenkins
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

## 4. Pipeline Configuration
1. **Create a Freestyle Job**
- On the **Jenkins dashboard**, click **New Item**.
- Enter the name `dev_deploy`, choose **Freestyle project**, and click **OK**.
2. **Configure the Job**
- In General:
  - Under **Source Code Management**, select **Git** and *provide the repository URL*.
  - Click **Apply & Save**.
  - Click **Build Now**
  - **Git File Structure:**
    - Ensure the repository contains
      - pom.xml
      - src (source code)
      - Dockerfile
        ```dockerfile
        FROM tomcat:9-jre9
        MAINTAINER "gyanaranjanmallick444@gmail.com"
        COPY ./live.war /usr/local/tomcat/webapps
        ```

## 5. Build Steps
### Step 5.1: Maven Build
- Add a Build Step:
- Under **Build Steps**, click **Add build step → Invoke top-level Maven targets**.
  - Set **Maven Version** to the version installed (e.g., `Maven`).
  - Set **Goals** to `clean package`.
  - Click **Apply & Save**.
  - *Trigger the build to generate the WAR file.*
    
### Step 5.2: Transfer WAR File to Ansible Node
- Add another Build Step:
- Under **Build Steps**, click **Add build step → Send files or execute commands over SSH.**
  - Set **SSH Server** to `Ansible`
  - Set **Transfer set** to `**/*.war`
  - Set **Remove prefix** to `target`
  - Click **Apply & Save**.
  - **Trigger the build to transfer the `WAR` file to the Ansible server**.
### Step 5.3: Transfer Dockerfile
- Add another Build Step:
- Under **Build Steps**, click **Add build step → Send files or execute commands over SSH.**
  - Set **SSH Server** to `Ansible`
  - Set **Transfer set** to `Dockerfile`
  - Click **Apply & Save**.
  - **Trigger the build to transfer the `Dockerfile` to the Ansible server.**

### Step 5.4: Build Docker Image
- 1. **In Dockerfile Build Step**
- In **Transfer set** to `Dockerfile`
- **Exec Command**:
  ```bash
  docker build -t mallick17/webproject .
  docker push mallick17/webproject
  ```
- Click **Apply & Save**.
-  **Trigger the build to  create `Dockerfile` Image and Push the image to Docker Hub.**
- 2. **Update the build step to use dynamic naming**
     ```bash
     docker build -t $JOB_NAME:v1.$BUILD_ID .
     docker tag $JOB_NAME:v1.$BUILD_ID mallick17/$JOB_NAME:v1.$BUILD_ID
     docker tag $JOB_NAME:v1.$BUILD_ID mallick17/$JOB_NAME:latest  //>> apply save >> build now
     docker push mallick17/$JOB_NAME:v1.$BUILD_ID
     docker push mallick17/$JOB_NAME:latest   //>> apply save >> build now
     docker rmi -f $JOB_NAME:v1.$BUILD_ID
     docker rmi -f mallick17/$JOB_NAME:latest   //>> apply save >> build now
     ```
  - Click **Apply & Save**.
### Step 5.5: Clean Docker Environment
- Ensure no containers or images with the same name exist
  ```
  docker rmi -f $(docker images -q)  ##deletes all the images
  ```
## 6. Create Ansible Playbook
 On the Master node, create a playbook file (`devtask.yml`) for container management:
 - 6.1. Create an Ansible playbook on the master node:
 ```bash
 vi /home/ansible/task.yml
```
**Playbook Content**
```yaml
---
- name: webapp playbook
  hosts: mallick
  user: ansible
  become: yes
  connection: ssh
  tasks:
    - name: remove image
      command: docker rmi -f mallick17/webproject:latest
    - name: remove container
      command: docker rm -f webproject
    - name: run a container
      command: docker run -it -d --name webapp -p 8090:8080 mallick17/webproject:latest
```

## 7. Final Pipeline Execution Deploy with Ansible
- Add another Build Step
- Under **Build Steps**, click **Add build step → Send files or execute commands over SSH.**
  - Set **SSH Server** to `Ansible`
  - Set **Exec Command** to:
  ```bash
  ansible-playbook /home/ansible/task.yml
  ```
  - Click **Apply & Save**.
  - *Trigger the build to deploy the Docker container using the Ansible playbook.*
 
## 8. Access the Application
- Copy the **Public IPv4** address of the **Worker node**.
- Access the application on the browser:
  ```vbnet
  http://<Public IPv4>:8090/finance
  ```
  ---
  ## Summary
  - This deployment setup automates the process of:
    - Building the application using Jenkins.
    - Packaging it into a Docker image.
    - Deploying the container using Ansible.
  - It provides a robust CI/CD pipeline for consistent and automated deployment.

  #### Additional Notes
    - **Permissions Issue with Docker:** <br>
     If you encounter a permissions error with the Docker socket, run:
    ```bash
    sudo chmod 777 /var/run/docker.sock
    ```
