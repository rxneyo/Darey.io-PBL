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

   ![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/a5beec1d-d32b-467d-9743-177bee3ac8b7)

   
2. After merging the PR, go back into your terminal and switch into the main branch.


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/bb61fb63-bede-48c2-b0d3-8b9e71b16bb3)


3. Pull the latest change.


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/c58478d4-f525-4bb1-a50c-cd735587fe1c)


4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)

 ![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/3fe22541-3bfb-4ad9-be27-b71f7dcaa51d)


   1. Package 

   2. Deploy 

   3. Clean up

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

        stage('Package') {
            steps {
                script {
                    sh 'echo "Packaging Stage"'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh 'echo "Deploying Stage"'
                }
            }
        }

        stage('Clean up') {
            steps {
                script {
                    sh 'echo "Cleaning up Stage"'
                }
            }
        }
    }
}
```
      

5. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/51ad5ee2-5f2d-4d2b-890d-170227d01fa4)


6. Eventually, your main branch should have a successful pipeline like this in blue ocean


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/ba2f35fc-4f01-44e5-ae9e-dfaf40bf8f2c)



## RUNNING ANSIBLE PLAYBOOK FROM JENKINS


Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

1. Installing Ansible on Jenkins

2. Installing Ansible plugin in Jenkins UI

3. Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)

   Add the following code to the Jenkins file:

```python
pipeline {
  agent any

  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/.ansible.cfg"
    }

  stages {
      stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

      stage('Checkout SCM') {
         steps{
            git branch: 'main', url: 'https://github.com/rxneyo/ansible-config-mgt.git'
         }
       }

      stage('Prepare Ansible For Execution') {
        steps {
          sh 'echo ${WORKSPACE}' 
          sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/.ansible.cfg'  
        }
     }

      stage('Run Ansible playbook') {
        steps {
           ansiblePlaybook become: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/dev, playbook: 'playbooks/site.yml'
         }
      }

      stage('Clean Workspace after build') {
        steps{
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
        }
      }
   }

}
```

**Note:** Ensure that Ansible runs against the Dev environment successfully.

Possible errors to watch out for:

1. Ensure that the git module in `Jenkinsfile` is checking out SCM to `main` branch instead of `master` (GitHub has discontinued the use of Master due to Black Lives Matter.)

2. Jenkins needs to export the `ANSIBLE_CONFIG` environment variable. You can put the `.ansible.cfg` file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set

3. Enter this into the `.ansinle.cfg` file:

```python
timeout = 160
callback_whitelist = profile_tasks
log_path=~/ansible.log
host_key_checking = False
gathering = smart
ansible_python_interpreter=/usr/bin/python3
allow_world_readable_tmpfiles=true


[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ControlPath=/tmp/ansible-ssh-%h-%p-%r -o ServerAliveInterval=60 -o ServerAliveCountMax=60 -o ForwardAgent=yes
```

**Possible issues to watch out for when you implement this:**

> Remember that `ansible.cfg` must be exported to environment variable so that Ansible knows where to find Roles. But because you will possibly run Jenkins from different git branches, the location of Ansible roles will change. Therefore, you must handle this dynamically. You can use Linux `Stream Editor` sed to update the section `roles_path` each time there is an execution. You may not have this issue if you run only from the main branch.

> If you push new changes to `Git` so that Jenkins failure can be fixed. You might observe that your change may sometimes have no effect. Even though your change is the actual fix required. This can be because Jenkins did not download the latest code from GitHub. Ensure that you start the `Jenkinsfile` with a clean up step to always delete the previous workspace before running a new one. Sometimes you might need to login to the Jenkins Linux server to verify the files in the workspace to confirm that what you are actually expecting is there. Otherwise, you can spend hours trying to figure out why Jenkins is still failing, when you have pushed up possible changes to fix the error.

> Another possible reason for Jenkins failure sometimes, is because you have indicated in the `Jenkinsfile` to check out the `main` git branch, and you are running a pipeline from another branch. So, always verify by logging onto the Jenkins box to check the workspace, and run git branch command to confirm that the branch you are expecting is there.

If everything goes well, it means, the `Dev` environment has an up-to-date configuration. But what if we need to deploy to other environments?

Are we going to manually update the `Jenkinsfile` to point inventory to those environments? such as `sit`, `uat`, `pentest`, etc.
Or do we need a dedicated git branch for each environment, and have the inventory part hard coded there.

Think about those for a minute and try to work out which one sounds more like a better solution.

Manually updating the `Jenkinsfile` is definitely not an option. And that should be obvious at this point. Because we try to automate things as much as possible.

Well, unfortunately, we will not be doing any of the highlighted options. What we will be doing is to parameterise the deployment. So that at the point of execution, the appropriate values are applied


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/9ac6bb6b-971d-470e-a628-a5e70440c543)


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/6c8b4c28-d228-41ac-a605-0d5cc61b11ab)

**Note:** I had terminated instances in the `dev.yml` file configurations. To test this playbook with servers, I had to spin up a new db server and adjusted necessary configurations to achieve this;

![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/dd45de11-73f6-46c3-91e5-9d40bc4a08e6)


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/1b2bc814-507c-4363-a722-c629cd212c6e)


![image](https://github.com/rxneyo/ansible-config-mgt/assets/125794122/f68e3b0d-f545-4fb8-b51c-0ae01cc55ea4)



## Parameterizing Jenkinsfile For Ansible Deployment


To deploy to other environments, we will need to use parameters.

1. Update `sit` inventory with new servers:

```python
[tooling]
<SIT-Tooling-Web-Server-Private-IP-Address>

[todo]
<SIT-Todo-Web-Server-Private-IP-Address>

[nginx]
<SIT-Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<SIT-DB-Server-Private-IP-Address>
```

2. Update `Jenkinsfile` to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution. It also has a description so that everyone is aware of its purpose:

```python
pipeline {
    agent any

    parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }
...
```

3. In the Ansible execution section, remove the hardcoded `inventory/dev` and replace with `inventory/${inventory}`
   
From now on, each time you hit on execute, it will expect an input

Notice that the default value loads up, but we can now specify which environment we want to deploy the configuration to. Simply type `sit` and hit Run

4. Add another parameter. This time, introduce `tagging` in Ansible. You can limit the Ansible execution to a specific role or playbook desired. Therefore, add an Ansible tag to run against `webserver` only. Test this locally first to get the experience. Once you understand this, update `Jenkinsfile` and run it from Jenkins.



## CI/CD PIPELINE FOR TODO APPLICATION


We already have `tooling` website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from `Artifactory` rather than from `git`. If you have not updated Ansible with an Artifactory role, simply use this guide to create an Ansible role for Artifactory (ignore the Nginx part). Configure Artifactory on Ubuntu 20.04 (https://www.howtoforge.com/tutorial/ubuntu-jfrog/)



## Phase 1 – Prepare Jenkins

1. Fork the repository below into your Github account

   `(https://github.com/rxneyo/php-todo.git)`

2. On you Jenkins server, install PHP, its dependencies and `Composer tool` (Feel free to do this manually at first, then update your Ansible accordingly later)


```python
sudo yum module reset php -y
sudo yum module enable php:remi-7.4 -y
sudo yum install -y php php-common php-mbstring php-opcache php-

intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm

#Install composer

curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/bin/composer
Verify Composer is installed or not
composer --version

#Install phpunit, phploc

sudo dnf --enablerepo=remi install php-phpunit-phploc
wget -O phpunit https://phar.phpunit.de/phpunit-7.phar
chmod +x phpunit
sudo yum install php-xdebug
```

3. Install Jenkins plugins

 1. `Plot plugin`

 2. `Artifactory plugin`

> We will use plot plugin to display tests reports, and code coverage information.

>The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.


   NB: Run the build with the ci inventory so it updates the artifactory server

I had to create a new `artifactory` role in `roles` using `ansible-galaxy role init artifactory`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/a989ea9b-1fd2-4895-a559-9299a7e545e6)


   ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/bb28e80f-a0a8-4087-a33b-a363d36534d1)


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/8858286a-645a-491e-861d-ab7e3d5ff72b)


**To confirm to go `public-ip:8081`. Login with the default credntials 'admin' and 'password' and then change the password. Then proceed to creating a generic local repository. It is required you open both port 8081 and 8082 in your inbound rules.**


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/95187406-c467-4222-a658-d46dcd23956b)


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/62fc74ba-2ac3-4af1-9839-dfe6cf4b0f71)


4. In Jenkins UI configure Artifactory

 ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/98e4bb23-e89c-4f95-bfcd-1c176f914c75)


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/1285ae3d-3619-44b6-abd7-646fe68cbd87)


Create a GENERIC Repository (i named mine rxneyo). This will be used to store our build artifacts

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/8453f23d-1a91-46ba-9c85-5df04d796a54)


## Phase 2 – Integrate Artifactory repository with Jenkins

1. In VsCode, Create a dummy `Jenkinsfile` in the to-do repository

2. Using Blue Ocean, create a multibranch Jenkins pipeline: Jankins dashboard > Open Blue Ocean > New Pipeline > Github > Select Repository (php-todo) > Create Pipeline > 

3. On the database server, create database and user


```python
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
```

I used this in `mysql` roles, in the `main.yml` of the defaults

```python
# Databases.
mysql_databases: 
   - name: tooling
     collation: utf8_general_ci
     encoding: utf8
     replicate: 1

mysql_databases:
  - name: homestead
    collation: utf8_general_ci
    encoding: utf8
    replicate: 1

# Users.
mysql_users:
  - name: webaccess
    host: 0.0.0.0
    password: secret
    priv: '*.*:ALL,GRANT'

mysql_users:
  - name: homestead
    host: db private ip
    password: sePret^i
    priv: '*.*:ALL,GRANT'
```

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/5333d5f2-a842-4601-a80b-71205bf4741a)



4. Update the database connectivity requirements in the file .env.sample

5. Update Jenkinsfile with proper pipeline configuration

```python
pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: '(https://github.com/rxneyo/php-todo.git)'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}
```


I got an error when running the playbook. The error came up because the Jenkins Server is the client server and it is unable to reach the DB server.


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/cbba41d6-9476-4ca4-8834-22da130f543b)


This was resolved by installing `mysql-client` on the jenkins server and updating the bind-address in the DB server:

`sudo yum install mysql -y`

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

Set bind address to 0.0.0.0


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/f6d3af60-e0ae-4b82-a96c-abfbe0b3768f)


> Update the database connectivity requirements in the file `.env.sample` file:

```python
DB_CONNECTION=mysql
DB_PORT=3306
```

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/e02d5111-d1e1-4a2e-a0dc-e019a3c8f287)






