# DEVOPS TOOLING WEBSITE SOLUTION


In previous [Project 6(https://github.com/rxneyo/Darey.io-PBL/blob/main/project6.md)) I implemented a WordPress based solution that is ready to be filled with content and can be used as a full fledged website or blog. Moving further we will add some more value to our solutions that your DevOps team could utilize. We want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.


The tools we want our team to be able to use are well known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:


1. `Jenkins` – free and open source automation server used to build CI/CD pipelines.

2. `Kubernetes` – an open-source container-orchestration system for automating computer application deployment, scaling, and management.

3. `Jfrog Artifactory` – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.

4. `Rancher` – an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.

5. `Grafana` – a multi-platform open source analytics and interactive visualization web application.

6. `Prometheus` – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.

7. `Kibana` – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.



##   Setup and technologies used in Project 7


As a member of a DevOps team, one will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

In this project I had to implement a solution that consists of following components:

1. **Infrastructure:** AWS
 
2. **Webserver Linux:** Red Hat Enterprise Linux 8


3. **Database Server:** Ubuntu 20.04 + MySQL

4. **Storage Server:** Red Hat Enterprise Linux 8 + NFS Server

5. **Programming Language:** PHP

6. **Code Repository:** GitHub


On the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it looks like a local file system from where they can serve the same files.



![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/2b5f2ab5-1339-4248-a363-98f9b101ee30)


It is very important to know what storage solution is suitable for what use cases, for this – you need to answer following questions: 

What data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. 

Based on anwers to the above questions, you will be able to choose the right storage system for your solution.


## STEP 1 – PREPARE NFS SERVER


Step 1 – Prepare NFS Server:


1. Spin up a new EC2 instance with RHEL Linux 8 Operating System.


2. Based on your LVM experience from Project 6, Configure LVM on the Server.

-Instead of formating the disks as `ext4` you will have to format them as `xfs`

`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`

`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`

`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/e294531f-5a37-4c35-9f6b-1bd0a730ab6e)



-Ensure there are **3 Logical Volumes**. `lv-opt` `lv-apps`, and `lv-logs`

`sudo lvcreate -n lv-apps -L 9G webdata-vg`

`sudo lvcreate -n lv-logs -L 9G webdata-vg`

`sudo lvcreate -n lv-apps -L 9G webdata-vg`


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/78174d50-3753-4d6f-885c-b8c4dc5fdb01)


Created Logical Volumes can be verified using;

`sudo lvs`

`lsblk`

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/444fd3ea-a318-426b-94b0-01ad17514855)


3. Create mount points on `/mnt` directory for the logical volumes as follow:

`sudo mkdir /mnt/apps`

`sudo mkdir /mnt/logs`

`sudo mkdir /mnt/opt`


-Mount `lv-apps` on `/mnt/apps` – To be used by webservers

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`


-Mount `lv-logs` on `/mnt/logs` – To be used by webserver logs

`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`


-Mount `lv-opt` on `/mnt/opt` – To be used by Jenkins server in Project 8

`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/3b285676-1554-419f-a71b-d94dc60131a9)



4. Install NFS server, configure it to start on reboot and make sure it is up and running

`sudo yum -y update`

`sudo yum install nfs-utils -y`

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/665f98e0-bc68-4244-bde5-2c2a1003a3cc)



5. Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security.

   
To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:


Next is to make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:


`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`


`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/logs`

`sudo chmod -R 777 /mnt/opt`


`sudo systemctl restart nfs-server.service`


Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):

`sudo vi /etc/exports`


`/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`

`/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`

`/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`


`Esc + :wq!`


`sudo exportfs -arv`


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/8c00bac9-0180-4d64-91be-6388275cc957)


6. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

`rpcinfo -p | grep nfs`

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/17af302c-5fc7-4e5f-b5d6-fa7d947b797a)


**Important note:** For NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/84f54e91-c37c-4584-8d58-d06cc9f4cce7)



## STEP 2 — CONFIGURE THE DATABASE SERVER


By now you should know how to install and configure a MySQL DBMS to work with remote Web Server

1. Install MySQL server

   `sudo apt update`
   
   `sudo apt install mysql-server -y`

   `sudo systemctl status mysql`

   ![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/273e1e4a-bcd7-4e25-a564-800e3d994802)



3. Create a database and name it `tooling`

   `sudo mysql`

   `mysql>` `create database tooling;`


5. Create a database user and name it `webaccess`

   `mysql>` `create user 'webaccess'@'(subnetIPv4CIDR)' identified by 'password';`

   
7. Grant permission to `webaccess` user on `tooling` database to do anything only from the webservers `subnet cidr`

`mysql>` `grant all privileges on tooling.* to 'webaccess'@'(subnetIPv4CIDR)';`

`mysql>` `flush privileges;`

Verify by;

`mysql>` `show databases`


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/0b8ab2d1-1edd-4c94-aa70-da7709542796)





## Step 3 — Prepare the Web Servers


We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.


You already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).


This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.


During the next steps we will do following:

* Configure NFS client (this step must be done on all three servers)

* Deploy a Tooling application to our Web Servers into a shared NFS folder

* Configure the Web Servers to work with a single MySQL database


1. Launch a new EC2 instance with RHEL 8 Operating System

2. Install NFS client

`sudo yum install nfs-utils nfs4-acl-tools -y`


3. Mount `/var/www/` and target the NFS server’s export for apps

`sudo mkdir /var/www`

`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`


4. Verify that NFS was mounted successfully by running `df -h`. Make sure that the changes will persist on Web Server after reboot:

`sudo vi /etc/fstab`


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/128c6cce-2278-4401-97fa-21430633fcc2)



add following line

`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`


5. Install Remi’s repository, Apache and PHP

   
`sudo yum install httpd -y`


`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`


`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`


`sudo dnf module reset php`


`sudo dnf module enable php:remi-7.4`


`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`


`sudo systemctl start php-fpm`


`sudo systemctl enable php-fpm`


`sudo setsebool -P httpd_execmem 1`



**Repeat steps 1-5 for another 2 Web Servers.**


6. Verify that Apache files and directories are available on the Web Server in `/var/www` and also on the NFS server in `/mnt/apps`. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.

  **NFS:** `ls /mnt/apps`

  ![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/cdb094c5-64a9-4381-8d38-241341dc64b6)


**Webserver:** `ls /var/www`

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/0679fd95-b29a-4079-a911-32741cc8c729)




   
7. Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №4 to make sure the mount point will persist after reboot.
   

   To locate log folder: `ls /var/log`

   ![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/67c8cc61-2d77-4012-9540-1010d5392c89)


   
8. Fork the tooling source code from Darey.io Github Account to your Github account.

   `sudo yum install git`

   `git init`

   `git clone https://github.com/darey-io/tooling.git`

   `ls`
   

   ![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/fd2e8350-8413-46d2-85d4-31e2e54ba6c9)

   


9. Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to `/var/www/html`

`sudo cp -R html/. /var/www/html`

Verify by `ls /var/www/html`  **OR** `ls html`

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/1d7d711a-d384-4caf-a838-7e44de04ef7e)



**Note 1:** Do not forget to open TCP port 80 on the Web Server.

**Note 2:** If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0


To make this change permanent – open following config file `sudo vi /etc/sysconfig/selinux` and set `SELINUX=disabled` then restart httpd.


10. Update the website’s configuration to connect to the database (in `/var/www/html/functions.php` file). Apply `tooling-db.sql` script to your database using this command

`mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

Before running the above command, Install mysql in the `tooling` directory:

`cd tooling`

`sudo yum install mysql -y`

**NOTE:** It is important to change the bind address of the **mysql conf** in the database server. This can be done by running the command below;

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

Then change `bind-address` and `mysqlx-bind-address  to 0.0.0.0

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/c949ba1f-6e99-44da-8674-88a495a73c77)


Then restart mysql;

`sudo systemctl restart mysql`

Then retry the command;

`mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

This time it will return no error. 

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/712a31f6-8ae4-47b7-b5fe-1ed93a841a18)


    
11. Create in MySQL a new admin user with username: `myuser` and password: `password`


INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);

This is done by;

`sudo mysql`

`mysql> show databases;`

`mysql> use tooling;`

`mysql> show tables;`

`mysql> select * from users;`

`

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/a3de34ae-1906-4856-af75-ac9019a5060e)



![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/68b1337c-5539-464c-95a2-4f946d48fbc1)



Open the website in your browser `http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php` and make sure you can login into the website with `myuser` user.


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/3e5d8ff0-ac1b-4c07-bcf2-62103953fe14)


Created a sign up process by adding user (myuser) credentials manually to the database

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/411bbc27-81b5-4a97-b97d-bd1bbf8f9e20)



Log into newly created `myuser` by running <public_ip_address>/login.php. on the browser 

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/dde7762c-6788-46ba-a85f-1aa9ab61f533)



