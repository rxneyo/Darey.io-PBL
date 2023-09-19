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


