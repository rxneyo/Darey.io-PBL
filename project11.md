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

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/3eac5ac1-2a05-4b9c-b050-265febb47fb1)


## BEGIN ANSIBLE DEVELOPMENT

In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

`git checkout -b >projectname<`

Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)

> Checkout the newly created feature branch to your local machine and start building your code and directory structure

> Create a directory and name it playbooks – it will be used to store all your playbook files.

 `mkdir playbooks` or use the add directory tool on vscode

> Create a directory and name it inventory – it will be used to keep your hosts organised.

`mkdir inventory`

> Within the playbooks folder, create your first playbook, and name it common.yml

> Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

`cd inventory`

`touch dev.yml staging.yml uat.yml prod.yml`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/9704cae2-5d6d-4d86-99f5-d656cd080beb)


## Step 4 – Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

Save below inventory structure in the `inventory/dev` file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:

`eval `ssh-agent -s`

`ssh-add <path-to-private-key>`

Confirm the key has been added with the command below, you should see the name of your key

`ssh-add -l`

Now, ssh into your Jenkins-Ansible server using ssh-agent

`ssh -A ubuntu@public-ip`

Also notice, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

Update your `inventory/dev.yml` file with this snippet of code:

[nfs]

`<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'`

[webservers]

`<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'`

`<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'`

[db]

`<Database-Private-IP-Address> ansible_ssh_user='ec2-user'`

[lb]

`<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/49b0af79-edad-4fc2-9b19-707af5beb448)



## Step 5 – Create a Common Playbook

It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in `inventory/dev`.

In `common.yml` playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your `playbooks/common.yml` file with following code:


`---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest`


Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

Feel free to update this playbook with following tasks:

> Create a directory and a file inside it

> Change timezone on all servers

> Run some shell script

> …
 
![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/2f14df93-d5c0-4575-829b-63f136c19f5b)



## Step 6 – Update GIT with the latest code

Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.

In the real world, you will be working within a team of other DevOps engineers and developers. It is important to learn how to collaborate with help of GIT. In many organisations there is a development rule that do not allow to deploy any code before it has been reviewed by an extra pair of eyes – it is also called "Four eyes principle".

Now you have a separate branch, you will need to know how to raise a Pull Request (PR), get your branch peer reviewed and merged to the master branch.

Commit your code into GitHub:

1. Use git commands to add, commit and push your branch to GitHub.

`git status`

`git add <selected files>`

`git commit -m "commit message"`


2. Create a Pull request (PR)

   ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/e1a36f97-11c4-48ce-950d-0f1ca92d172d)

   ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/2f1880d3-e531-436f-83a8-320cb7bc2ca7)
![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/383991ef-5d32-43c1-9c5b-fe155f24f9b0)



4. Wear a hat of another developer for a second, and act as a reviewer.

5. If the reviewer is happy with your new feature development, merge the code to the master branch.

6. Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/77b96db0-efb5-4d6e-90f9-577ff6bf1571)


Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/3e693ebb-dda2-488d-9c79-0b7de2a85b6a)



## Step 7 – Run first Ansible test

Now, it is time to execute `ansible-playbook` command and verify if your playbook actually works:

`cd ansible-config-mgt`

`ansible-playbook -i inventory/dev.yml playbooks/common.yml`

You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version

ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml
