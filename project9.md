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


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/c71b8eb3-22db-4430-a4cd-161b87f3b1fd)


2. Go to Jenkins web console, click “New Item” and create a “Freestyle project”

On the jenkins server, create a new freestyle job


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/6772f260-756c-4709-bd32-2640c229b803)


To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

In doing the above, choose Git repository, the provide the link to the GitHub tooling repository and credentials (username and password. This is to allow Jenkins to access files in the repository. Also specify the branch containing code; it could be main or master branch.

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/7c712d63-893e-4629-b68b-ba305fddf385)

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/f6bd4efa-b637-49aa-ad1d-93cca8ea8899)

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/60915207-0058-4a58-a3dd-b532124ce353)


Save the configuration and let us try to run the build. For now we can only do it manually.


Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under #1

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/e8667897-a80d-4ec9-b1d3-e7cc05447277)

**NB:** My Build 1 was wrong due to wrong branch specification. Correcting it made Build 2 successful as shown in the Console Output below:

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/15b40c9d-3eef-4a07-893e-fe7e01b92382)


Also note that this build does not produce anything and it runs only when we trigger it manually. Let us fix it.


3. Click “Configure” your job/project and add these two configurations

   This is necessary to specify the particular trigger to use for triggering the job.

   
Configure triggering the job from GitHub webhook:

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/5b07c6f2-1401-4be8-9901-cdae50015ac8)

Next is to configure “Post-build Actions” to archive all the files – files resulted from a build are called “artifacts”.

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/1abb84e4-bbd8-4212-ab61-403ba0cdc1dd)


Now, we can go ahead to make some changes in any file in our GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/b4b71438-05c6-443b-9eed-d54dcbc761e8)


The console output shows the created job and the successful build. In this case the code on Github was built into an artifact on our Jenkins server workspace. It is possible to find the artificat by checking the status tab of the completed job 

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/f2bf5990-4e58-4fc6-bb13-7b36f4a52577)

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/ab31180e-124a-4db8-918f-9109c475f5ac)


Now we have configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.


By default, the artifacts are stored on Jenkins server locally. Our created artifact can be found on our local terminal too at this path 

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/660436e2-8340-42a5-a48f-92b96a8a9005)



## CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH

## Step 3 – Configure Jenkins to copy files to NFS server via SSH

Now that we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to `/mnt/apps` directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called “Publish Over SSH”. To achieve this, we install the Publish Via SSH pluging on Jenkins. The plugin allows one to send newly created packages to a remote server and install them, start and stop services that the build may depend on and many other use cases.

1. Install “Publish Over SSH” plugin.


On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item.


On “Available” tab search for “Publish Over SSH” plugin and install it 

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/33de7e69-d0b4-4871-94e2-56ee73a44f71)


2. Configure the job to copy artifacts over to NFS server. On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to the NFS server:

Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

Hostname – can be private IP address of NFS server
Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/d1d67116-9263-496b-9dce-7fd7b174208c)


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/e617e1ff-0095-48b3-a166-a4debb98b7fb)


Now make a new change on the tooling source code and push to github, Jenkins will build an artifact by downloading the code into its workspace based on the latest commit and via SSH it publishes the artifact into the NFS Server to update the source code.

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/50e1c06e-5f2c-4000-9574-f53634397c7c)


To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file

`cat /mnt/apps/README.md`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/8326b5fa-1d3c-48c3-8613-1c7044377356)



If you see the changes you had previously made in your GitHub – the job works as expected.
