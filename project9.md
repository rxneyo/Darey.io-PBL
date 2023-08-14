# TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

## Introduction

In previous Project 8 we introduced horizontal scalability concept, which allow us to add new Web Servers to our Tooling Website and you have successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. If it is just two or three servers – it is not a big deal to configure them manually. Imagine that you would need to repeat the same task over and over again adding dozens or even hundreds of servers

In this project we are going to start automating part of our routine tasks with a free and open source automation server – Jenkins. 


It is one of the most popular CI/CD tools. Jenkins is an open-source Continuous Integration server written in Java for setting up a chain of actions to achieve the Continuous Integration process in an automated fashion. Jenkins supports the complete development life cycle of software from building, testing, documenting the software, deploying, and other stages of the software development life cycle.


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/4bbec492-8aca-49b2-9740-d15e28b366c2)


## Task


The task for this project is to enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

In summary; this project is to introduce how to use Jenkins to automate code delivery to the NFS server

Here is how your updated architecture will look like upon competition of this project:


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/61d6464c-67d8-4ad6-b7c5-cb68da7f025f)


## INSTALL AND CONFIGURE JENKINS SERVER


## Step 1 – Install Jenkins server


1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins”


2. Install JDK (since Jenkins is a Java-based application)

`sudo apt update`

`sudo apt install default-jdk-headless`


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/71e3198d-630d-44ce-b344-2bf42ca2cac9)



3. Install Jenkins

`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`

`sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \`
    /etc/apt/sources.list.d/jenkins.list'

`sudo apt update`

`sudo apt-get install jenkins`


Make sure Jenkins is up and running


`sudo systemctl enable jenkins`

`sudo systemctl start jenkins`

`sudo systemctl status jenkins`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/8fa20c1a-951c-4cc3-be06-c0a044e94087)



4. By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

 Since Jenkins runs on default port 8080, port 8080 should be opened on the Security Group inbound rule of the jenkins server on AWS as shown below

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/928465ef-892b-477c-aa9d-77cc331cd3e6)


5. Perform initial Jenkins setup.
   
From your browser access `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080` as shown below;

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/8666c20a-82cf-48e2-809b-3a698858bb7a)



There will be a prompt to provide a default admin password. The password can be retrieved from the server by running this command:


`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`


The password is in the /var/lib/jenkins/secrets/initialAdminPassword path on the server.


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/c684c874-9d20-4a8b-b2b0-ca2f1261a5ee)


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/739aca73-7d46-45e1-9c48-67df216c254c)


Once plugins installation is done – create an admin user and you will get your Jenkins server address.

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/cc51d71a-f80c-4998-b0aa-1b63a95de9ee)



## Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks


In this part, the goal is to learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.


1. Enable webhooks in your GitHub repository settings

On the github repository that contains application code, create a webhook to connect to the jenkins job. To create webhook, go to the settings tab on the github repo and click on webhooks. Webhook should look like this

