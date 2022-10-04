# TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION INTRODUCTION TO JENKINS

This project will make use of Jenkins for Continuous Inteegration to automatically checkout code from version control and integrate it into the NFS server which will be made available to all the hosts client server.

## Project Requirements
- Amazon EC2 Instances (RedHat OS)
- Amazon EC2 instance to serve as Jenkins server
- Amazon EC2 Instance running Ubuntu OS to serve as our load balancer
- An existing three-tier achitecture to serve as web server and database server


![Project Architecture Diagram](https://darey.io/wp-content/uploads/2021/07/add_jenkins.png)

### Jenkins Server Setup
- Launch an Ubuntu Server 20.04
- Allow traffic on port 8080 for HTTP and 22 for SSH connection 

**Installing and configuring Jenkins**

Jenkins required Java development kit to work.

```bash
    sudo apt update -y
    sudo apt install -y default-jdk-headless

    #Download and Install Jenkins
    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
        /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt-get install jenkins

    #Make sure Jenkins is up and running
    sudo systemctl status jenkins
```

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-jenkins-status.png)

**Access Jenkins UI**
- Open your browser
- http://JENKINS-SERVER-ADDRESS:8080

- Copy the default admin password

```
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-jenkins-ui.png)

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-jenkins-user-creation.png)

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-jenkins-plugin.png)

**Create Jenkins job (Freestyle Project)**

From the Jenkins web console, click "New Item" and create a "Freestyle project"
Install **Publish over ssh** plugin so that the project can be copied to the NFS server.

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-plugins.png)

Configure the job/project to copy artifacts over to NFS server.

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-publish-over-ssh.png)

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-jenkins-ssh.png)

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-successful-build.png)

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-09/screenshots/project9-jenkins-pipeline-ssh-testing.png)

