# ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10


## Ansible Client as a Jump Server (Bastion Host)


A Jump Server (also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provides better security and reduces attack surface.


In the diagram below the Virtual Private Network (VPC) is divided into two subnets – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/9ea4bc33-3abc-4518-8cd6-0e7d8e31b8d4)


In this project, we will develop Ansible scripts to simulate the use of a Jump box/Bastion host to access our Web Servers.

## Task

1. Install and configure Ansible client to act as a Jump Server/Bastion Host

2. Create a simple Ansible playbook to automate servers configuration





## INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE


1. Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

 ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/d4184e84-64d6-43ac-865d-47b961f429cc)



3. In your GitHub account create a new repository and name it ansible-config-mgt.


4. Install Ansible

`sudo apt update`

`sudo apt install ansible`

Check your Ansible version by running `ansible --version`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/024ff349-ff90-4378-ac44-043c81181cdd)



 4. Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.

  > Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
  
  >Configure Webhook in GitHub and set webhook to trigger ansible build.
  
  >Configure a Post-build job to save all (**) files, like you did it in Project 9.


5. Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/4e0a70ad-458b-495d-8318-ef0087764f6b)

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/01b3f1ac-2214-4a06-8542-ba489527095c)

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/86d3ea88-f9a9-4370-a898-b3aba4adda54)



Note: Trigger Jenkins project execution only for /main (master) branch.

Now your setup will look like this:

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/f4e8c5c8-152a-4b2d-be5c-d08e02fb39dc)


Tip: Every time you stop/start your Jenkins-Ansible server – you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to your Jenkins-Ansible server (you have done it before to your LB server in Project 10). Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.



## Step 2 – Prepare your development environment using Visual Studio Code

1. First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSCode), downloading it is essential.

2. After you have successfully installed VSC, configure it to connect to your newly created GitHub repository (ansible-config-mgt)

3. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

`git clone <ansible-config-mgt repo link>`

