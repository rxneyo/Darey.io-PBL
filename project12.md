# Jenkins CI/CD on a 3-tier application && Ansible Configuration Management Dev and UAT servers using Static Assignments


## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project we will continue working with `ansible-config-mgt` repository and make some improvements of our code. here we will learn how to refactor Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.


In the previous project, I implemented CI/CD and Configuration Managment solutions on the Development Servers using Ansible. In this project, I will be extending the functionality of this architecture and introducing configurations for UAT environment.


## STEP 1 - Jenkins Job Enhancement

1. Go to your `Jenkins-Ansible` server and create a new directory called `ansible-config-artifact` – we will store there all artifacts after each build.
   
`sudo mkdir /home/ubuntu/ansible-config-artifact`


2. Change permissions to this directory, so Jenkins could save files there –

 `sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact`


3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on `Available` tab search for `Copy Artifact` and install this plugin without restarting Jenkins


   ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/b62b6817-5b6c-4389-ac4f-01d3a4ca3a8f)


4. Create a new Freestyle project (you have done it in Project 9) and name it `save_artifacts`

   
5. This project will be triggered by completion of your existing ansible project. Configure it accordingly:

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/a8683ca2-e5ba-4b8d-91a0-29881e59103e)

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/55967b5a-38c1-4e0f-9f60-2ca84eef37ca)


I configured the number of build to 5. This is useful because whenever the jenkins pipeline runs, it creates a directory for the artifacts and it takes alot of space. By specifying the number of builds, we can choose to keep only 5 of the latest builds and discard the rest.


6. The main idea of `save_artifacts` project is to save artifacts into `/home/ubuntu/ansible-config-artifact` directory. To achieve this, create a `Build` step and choose `Copy artifacts from other project`, specify `ansible` as a source project and `/home/ubuntu/ansible-config-artifact` as a target directory

 ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/7d4a4d11-ae15-4279-9e50-ca4feebf4a6a)


7. Test your set up by making some change in README.MD file inside your `ansible-config-mgt` repository (right inside master branch).

   
If both Jenkins jobs have completed one after another – you shall see your files inside `/home/ubuntu/ansible-config-artifact` directory and it will be updated with every commit to your `master` branch.


Now your Jenkins pipeline is more neat and clean.


## Step 2 – Refactor Ansible code by importing other playbooks into site.yml

Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it `refactor`.

In Project 11 , I wrote all tasks in a single playbook `common.yml`, now it is pretty simple set of instructions for only 2 types of OS, but imagine you have many more tasks and you need to apply this playbook to other servers with different requirements. In this case, you will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook.

Let see code re-use in action by importing other playbooks.

1. Within `playbooks` folder, create a new file and name it `site.yml` – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, `site.yml` will become a parent to all other playbooks that will be developed. Including `common.yml` that you created previously. Dont worry, you will understand more what this means shortly.

2. Create a new folder in root of the repository and name it `static-assignments`. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.

3. Move `common.yml` file into the newly created `static-assignments` folder.

4. Inside `site.yml` file, import `common.yml` playbook.

`---`
`- hosts: all`
`- import_playbook: ../static-assignments/common.yml`

The code above uses built in `import_playbook` Ansible module.

Your folder structure should look like this;

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/51799bec-91c2-4533-8038-5c90e5cc01e7)


5. Run `ansible-playbook` command against the dev environment

   
Since you need to apply some tasks to your `dev` servers and `wireshark` is already installed – you can go ahead and create another playbook under `static-assignments` and name it `common-del.yml`. In this playbook, configure deletion of `wireshark` utility.

`---`
`- name: update web, nfs and db servers`
`  hosts: webservers, nfs, db`
`  remote_user: ec2-user`
`  become: yes`
`  become_user: root`
`  tasks:`
`  - name: delete wireshark`
`    yum:`
`      name: wireshark`
`      state: removed`
`
`- name: update LB server`
`  hosts: lb`
`  remote_user: ubuntu`
`  become: yes`
`  become_user: root`
`  tasks:`
`  - name: delete wireshark`
`    apt:`
`      name: wireshark-qt`
`      state: absent``
`      autoremove: yes`
`      purge: yes`
`      autoclean: yes`

    We update `site.yml` with `- import_playbook:` `../static-assignments/common-del.yml` instead of `common.yml` and run it against `dev` servers

    cd /home/ubuntu/ansible-config-mgt/


ansible-playbook -i inventory/dev.yml playbooks/site.yaml
