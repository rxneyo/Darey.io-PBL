# WEB SOLUTION WITH WORDPRESS


In this project, the task is to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).


Project 6 consists of two parts:


1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.


2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

As a DevOps engineer, deep understanding of core components of web solutions and ability to troubleshoot them will play essential role in further progress and development.


## Three-tier Architecture


Generally, web, or mobile solutions are implemented based on what is called the `Three-tier Architecture`.


`Three-tier Architecture` is a client-server software architecture pattern that comprise of 3 separate layers.


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/5d5a3910-3df3-407b-bf08-23db927d0eae)



1. `Presentation Layer` (PL): This is the user interface such as the client server or browser on our laptops.


2. `Business Layer` (BL): This is the backend program that implements business logic. Application or Webserver


3. `Data Access or Management Layer` (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server



In this project, I had hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.


`Note`: This project gradually introduces new AWS elements into our solutions.  


## 3-Tier Setup:

1. A Laptop or PC to serve as a client

2. An EC2 Linux Server as a web server (This is where to install WordPress)

3. An EC2 Linux server as a database (DB) server


   
**Use RedHat OS for this project**


# LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.


## Step 1 — Prepare a Web Server

A web server should be prepared by following the steps below;

1. Launch an EC2 instance that will serve as “Web Server”. Create 3 volumes in the same Availability Zone as the Web Server EC2, each of 10 GiB.

 
 Here's a sample: ![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/71320acf-9188-43f0-8481-a2dad82065a0)


2. Attach all three volumes one by one to your Web Server EC2 instance as below;

   ![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/6f2c91c8-727f-405e-9f0f-406ec8fefe06)


Next is to open up the Linux terminal to begin configuration.

3. On the terminal, use `lsblk` command to inspect what block devices are attached to the server. Notice names of the newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with `ls /dev/` and make sure all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.


4. Use `df -h` command to see all mounts and free space on the server

![df -h](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/3fd45b44-6490-4911-bf1b-7e7de182afcd)


5. Use gdisk utility to create a single partition on each of the 3 disks


`sudo gdisk /dev/xvdf`


![sudo gdisk](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/b1e6ba3d-9a46-4b0f-be60-1200cdd4492e)
![sudo gdisk2](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/f44d55b8-adb1-4597-9eb7-11c0fae4094a)


6. Use lsblk utility to view the newly configured partition on each of the 3 disks.

![lsblk2](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/b0b50833-dd49-4d34-aa0e-df112ac4021f)


7. Install **lvm2** package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.

   ![lvm2 package installed](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/33e1ec18-db48-4958-9172-867045b6e657)
   ![lvmdiskscan](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/8afea757-7878-49a8-9407-06d8f45fae6b)


**Note:** Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used. So we shall use yum command instead.


8. Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/xvdf1`

`sudo pvcreate /dev/xvdg1`

`sudo pvcreate /dev/xvdh1`


9. We need to verify that our Physical volume has been created successfully by running `sudo pvs`

![sudo pvs](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/f6848307-7858-4a24-b6da-10f8af78fefb)


10. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg**

   
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`


11. Verify that our VG has been created successfully by running `sudo vgs`


![vgs](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/7ebf995b-6d14-4a01-8edc-7ff904679c7c)


12. Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size.
 
**NOTE:** apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.


`sudo lvcreate -n apps-lv -L 14G webdata-vg`


`sudo lvcreate -n logs-lv -L 14G webdata-vg`


13. Verify that our Logical Volume has been created successfully by running `sudo lvs`


14. Verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`


`sudo lsblk`

![View entire setup](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/9ade4322-8a13-4562-9c68-00350fcb4fac)


--- Use `mkfs.ext4` to format the logical volumes with `ext4` filesystem


`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`


`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![mkfs ext4 app-l](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/e9b73600-1f5b-4c49-8414-2c379ed21436)
![mkfs ext4 logs-lv](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/c86b3c57-0fea-4171-b636-fab0e95273cf)



15. Create `/var/www/html` directory to store website files


`sudo mkdir -p /var/www/html`


16. Create `/home/recovery/logs` to store backup of log data

`sudo mkdir -p /home/recovery/logs`


17. Mount `/var/www/html` on `apps-lv` logical volume


`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`


18. Use `rsync` utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** *(This is required before mounting the file system)*


`sudo rsync -av /var/log/. /home/recovery/logs/`


19. Mount **/var/log** on **logs-lv** logical volume. *(Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)*


`sudo mount /dev/webdata-vg/logs-lv /var/log`

![mount var webdata](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/51b0ba04-e034-45a2-8cf0-be19a7deba02)


20. Restore log files back into **/var/log** directory


`sudo rsync -av /home/recovery/logs/log/. /var/log`


21. Update `/etc/fstab` file so that the mount configuration will persist after restart of the server.


The UUID of the device will be used to update the `/etc/fstab` file;


`sudo blkid`


![blkid](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/86a02a24-f862-4376-a951-bf64c90db109)


`sudo vi /etc/fstab`


Update `/etc/fstab` in this format using your own UUID and rememeber to remove the leading and ending quotes.



22. Test the configuration and reload the daemon
    

`sudo mount -a`


`sudo systemctl daemon-reload`


23. Verify the setup by running df -h, output must look like this:
    

![fstab df h](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/602e9c91-caba-4cc5-8430-d11c69600427)




# PREPARE THE DATABASE SERVER

## Step 2 — Prepare the Database Server


Launch a second RedHat EC2 instance and tag it – ‘DB Server’


Repeat the same steps as for the Web Server, but instead of `apps-lv` create `db-lv` and mount it to `/db` directory instead of `/var/www/html/`.


## Step 3 — Install WordPress on your Web Server EC2


1. Update the repository


`sudo yum -y update`


2. Install wget, Apache and it’s dependencies


`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`


3. Start Apache


`sudo systemctl enable httpd`


`sudo systemctl start httpd`


4. To install PHP and it’s dependencies


`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo yum module list php`

`sudo yum module reset php`

`sudo yum module enable php:remi-7.4`

`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`setsebool -P httpd_execmem 1`


5. Restart Apache

   
`sudo systemctl restart httpd`


6. Download wordpress and copy wordpress to `var/www/html`

   
  `mkdir wordpress`
  
  `cd   wordpress`
  
  `sudo wget http://wordpress.org/latest.tar.gz`
  
  `sudo tar xzvf latest.tar.gz`
  
  `sudo rm -rf latest.tar.gz`
  
  `cp wordpress/wp-config-sample.php wordpress/wp-config.php`
  
  `cp -R wordpress /var/www/html/`


7. Configure SELinux Policies
   
  `sudo chown -R apache:apache /var/www/html/wordpress`
  
  `sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`
  
  `sudo setsebool -P httpd_can_network_connect=1`


## Step 4 — Install MySQL on your DB Server EC2


`sudo yum update`

`sudo yum install mysql-server`


Verify that the service is up and running by using `sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:


`sudo systemctl restart mysqld`

`sudo systemctl enable mysqld`


![mysql database running](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/a67dd65b-202e-4d5c-8f3e-b9b64c9ee3d6)



## Step 5 — Configure DB to work with WordPress

`sudo mysql`

`CREATE DATABASE wordpress;`

`CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';`

`GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';`

`FLUSH PRIVILEGES;`

`SHOW DATABASES;`

`exit`


## Step 6 — Configure WordPress to connect to remote database.

**Hint:** Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, allow access to the DB server ONLY from Web Server’s IP address, so in the Inbound Rule configuration specify source as /32 as displayed below

![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/327c4c08-5f87-483d-a7da-525355e269aa)



1. Install MySQL client and test that you can connect from your Web Server to your DB server by using `mysql-client`


`sudo yum install mysql`


`sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`


2. Verify if you can successfully execute `SHOW DATABASES;` command and see a list of existing databases.


3. Change permissions and configuration so Apache could use WordPress:

4. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)


5. Try to access from your browser the link to your WordPress `http://<Web-Server-Public-IP-Address>/wordpress/`


![webserver public ip fine](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/479de65f-049b-4560-a445-eb25ebe107a3)






![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/e8d45fba-bf2f-4768-9533-5586d22dc817)
