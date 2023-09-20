# EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP


# CI/CD PIPELINE FOR A PHP BASED APPLICATION

## Project Description:

As part of the ongoing infrastructure development with Ansible started from Project 11, in this task, I will be creating a pipeline that simulates continuous integration and delivery. Target end to end CI/CD pipeline is represented by the diagram below. It is important to know that both Tooling and TODO Web Applications are based on an interpreted (scripting) language (PHP). It means, it can be deployed directly onto a server and will work without compiling the code to a machine language.

The problem with that approach is, it would be difficult to package and version the software for different releases. And so, in this project, we will be using a different approach for releases, rather than downloading directly from git, we will be using Ansible `uri module`.


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/dd8b27d2-cdfe-4e09-8527-1df627807e15)

In this project, I will be setting up a CI/CD Pipeline for a PHP based application. The overall CI/CD process looks like the architecture above.

This project is architected in two major repositories with each repository containing its own CI/CD pipeline written in a Jenkinsfile

1. **ansible-config-mgt REPO**: this repository contains JenkinsFile which is responsible for setting up and configuring infrastructure required to carry out processes required for our application to run. It does this through the use of ansible roles. This repo is infrastructure specific

2. **PHP-todo REPO**: this repository contains jenkinsfile which is focused on processes which are application build specific such as building, linting, static code analysis, push to artifact repository etc


## Prerequisites

Will be making use of AWS virtual machines for this and will require 6 servers for the project which includes: 

Nginx Server: This would act as the reverse proxy server to our site and tool.

Jenkins server: To be used to implement your CI/CD workflows or pipelines. Select a t2.medium at least, Ubuntu 20.04 and Security group should be open to port 8080

SonarQube server: To be used for Code quality analysis. Select a t2.medium at least, Ubuntu 20.04 and Security group should be open to port 9000

Artifactory server: To be used as the binary repository where the outcome of your build process is stored. Select a t2.medium at least and Security group should be open to port 8081

Database server: To server as the databse server for the Todo application

Todo webserver: To host the Todo web application.

This project is partly a continuation of previous Ansible work, so simply add and subtract based on the new setup in this project. It will require a lot of servers to simulate all the different environments from dev/ci all the way to production. This will be quite a lot of servers altogether (But I don’t have to create them all at once. here, i'll only create servers required for an environment i'm working with at the moment. For example, when doing deployments for development, there is no need to create servers for integration, pentest, or production yet).

NB: Going forward, try to utilize your AWS free tier as much as you possible to reduce expenses.


To get started, we will focus on these environments initially.

* Ci
* Dev
* Pentest

Both `SIT` – For System Integration Testing and `UAT` – User Acceptance Testing do not require a lot of extra installation or configuration. They are basically the webservers holding our applications. But `Pentest` – For Penetration testing is where we will conduct security related tests, so some other tools and specific configurations will be needed. In some cases, it will also be used for `Performance and Load testing`. Otherwise, that can also be a separate environment on its own. It all depends on decisions made by the company and the team running the show.


What we want to achieve, is having Nginx to serve as a reverse proxy for our sites and tools. Each environment setup is represented in the below table and diagrams.


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/f5dfb6a0-65da-4df9-88bd-03414d146769)


The CI Environment:

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/f2238fa9-2c08-4c43-90f3-6f19971fe6dd)


Other Environments from Lower to Higher:

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/d2b69017-635f-4244-8992-95d8486c3035)


DNS requirements:

Make DNS entries to create a subdomain for each environment. Assuming your main domain is darey.io

You should have a subdomains list like this:


Server	Domain

Jenkins	https://ci.infradev.darey.io

Sonarqube	https://sonar.infradev.darey.io

Artifactory	https://artifacts.infradev.darey.io

Production Tooling	https://tooling.darey.io

Pre-Prod Tooling	https://tooling.preprod.darey.io

Pentest Tooling	https://tooling.pentest.darey.io



Ansible Inventory should look like this:

```python
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```

`ci` inventory file


```python
[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>
```

`dev` inventory file

```python
[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
```

`pentest` inventory file

```python
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>
```


**Observations:**

1. You will notice that in the pentest inventory file, we have introduced a new concept `pentest:children` This is because, we want to have a group called `pentest` which covers Ansible execution against both `pentest-todo` and `pentest-tooling` simultaneously. But at the same time, we want the flexibility to run specific Ansible tasks against an individual group.

2. The `db` group has a slightly different configuration. It uses a RedHat/Centos Linux distro. Others are based on ``Ubuntu (in this case user is `ubuntu`). Therefore, the user required for connectivity and path to python interpreter are different. If all your environment is based on Ubuntu, you may not need this kind of set up. Totally up to you how you want to do this. Whatever works for you is absolutely fine in this scenario.

 This makes us to introduce another Ansible concept called `group_vars`. With group vars, we can declare and set variables for each group of servers created in the inventory file.

For example, If there are variables we need to be common between both `pentest-todo` and `pentest-tooling`, rather than setting these variables in many places, we can simply use the `group_vars` for `pentest`. Since in the inventory file it has been created as `pentest:children` Ansible recognizes this and simply applies that variable to both children.

## ANSIBLE ROLES FOR CI ENVIRONMENT

To automate the setup of `SonarQube` and `JFROG` Artifactory, we can use `ansible-galaxy` to install this configuration into our ansible roles which will be used and run against the `sonarqube` server and artifactory server.

We will see this in play later


## Configuring Ansible For Jenkins Deployment


We create a Jenkins-server with a t2.medium specification because we will be needing more compute power to run builds compared to the jenkins-server we have been using in project 13

Prepare your Jenkins server:

Connect to your Jenkins instance on VScode via SSH and set up SSH-agent to ensure ansible get the private key required to connect to all other servers:

`eval `ssh-agent -s`

`ssh-add <path-to-private-key>`


Install the following packages and dependencies on the server:


1. Install git : `sudo yum install git` (using a rhel server)

2. Clone down the Asible-config-mgt repository: git clone (https://github.com/rxneyo/ansible-config-mgt.git)

3. Install Jenkins and its dependencies. Steps to install Jenkins can be found here:

 Install Java:  `sudo yum install java-11-openjdk`

 Enable the Jenkins repository on the server: `sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo`

 Import the Jenkins RPM GPG key: `sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key`

 Now install Jenkins: `sudo yum install jenkins`

 Then start the Jenkins service: `sudo systemctl start jenkins`
                                 `sudo systemctl enable jenkins`


5. Configure Ansible For Jenkins Deployment: `http://your_server_ip_or_domain:8080`


6. Navigate to Jenkins URL: :8080

In the Jenkins dashboard, click on Manage Jenkins -> Manage plugins and search for Blue Ocean plugin. Install and open Blue Ocean plugin.


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/bd838a13-b319-4287-8899-696d03696c59)


Create  new pipeline

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/3f4572f8-02df-47ba-bf89-7d0150601eb2)


Select Github

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/7a57e0ad-99a7-48e9-be33-3fe65a6634b8)


Connect Jenkins with Github


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/335b9343-f835-413f-a6ef-dde47a223be2)



Get personal access token from github

Copy access token

Paste the token and connect

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/eb4e1e34-50c6-49b1-bae4-9077defc1011)


Create a new pipeline

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/5c3bde6f-6370-446c-aaa3-d025abc73604)


At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.


Here is our newly created pipeline. Note that this pipeline takes the name of your GitHub repository.

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/c1a49777-b4b4-4f2d-a3f3-85061dbd159d)


This job gets created automatically by blue-ocean after connection with github repo.



## Creating JENKINSFILE


* In Vscode, inside the Ansible project, create a new directory and name it deploy, create a new file Jenkinsfile inside the directory.


 ![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/6aac9cf9-3398-49e3-bcda-2eff597f9647)



* Add the code snippet below to start building the `Jenkinsfile` gradually. This pipeline currently has just one stage called `Build` and the only thing we are doing is using the `shell script` module to echo `Building Stage`.

  ```python
  pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```

Now go back into the Ansible pipeline in Jenkins, and select configure

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/e06e8698-85c7-4dae-9d87-69bfd2e8b9e4)


Scroll down to `Build Configuration` section and specify the location of the Jenkinsfile at `deploy/Jenkinsfile`


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/c95e6aa4-bcdd-496e-b5bc-e522574835d1)

Back to the pipeline again, this time click "Build now"

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/38762103-a639-4456-a116-0c23f44e982e)


This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/ed01cc19-feca-46fb-852a-d0985c0fbba2)


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/85c56cb8-9f29-4c2b-ada2-7fde56d215ef)


To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

1. Click on Open Blue Ocean

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/e0c8d9d4-6322-4bc4-a936-d6e93cd69e65)


2. Select your project. Mine is `ansible-config-mgt`

3. Click on the play button against the branch

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/c175ff66-4628-41e8-815f-41a60cad667f)


Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

Let us see this in action.

1. Create a new git branch and name it `feature/jenkinspipeline-stages`

2. Currently we only have the `Build` stage. Let us add another stage called `Test`. Paste the code snippet below and push the new changes to GitHub.

```python
 pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```

To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

1. Click on the "Administration" button

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/c4857758-cabe-4d8f-8158-e2968f81077b)


2. Navigate to the Ansible project and click on "Scan repository now"

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/70ab8f49-ee3c-4652-bdc4-7805db530246)


3. Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too

   ![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/d6f973e8-a127-4eb6-a4a6-92f4e41da52e)


4. In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/e03e0574-2ae4-44c5-a7d1-d576b7adcfb4)


 ## A QUICK TASK FOR YOU!
 
1. Create a pull request to merge the latest code into the main branch
   
2. After merging the PR, go back into your terminal and switch into the main branch.

3. Pull the latest change.

4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
 
   1. Package 

   2. Deploy 

   3. Clean up
      

6. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch

7. Eventually, your main branch should have a successful pipeline like this in blue ocean
