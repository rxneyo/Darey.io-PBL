# Load Balancing With Apache


In Project-7 we had 3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server, let alone millions of Google servers.

In order to hide all this complexity and to have a single point of access with a single public IP address/name, a **Load Balancer** can be used. A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/88758171-272c-4a37-8ff1-37c93cd29458)


In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

## Task


Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. Make sure that users can be served by Web servers through the Load Balancer.

To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.


## Prerequisites


It is important to ensure that you have following servers installed and configured from Project-7:

1. Two RHEL8 Web Servers

2. One MySQL DB Server (based on Ubuntu 20.04)

3. One RHEL8 NFS server


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/f21d1e3f-2c33-4def-9263-4609c6455468)



# CONFIGURE APACHE AS A LOAD BALANCER

## Configure Apache As A Load Balancer

1. Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/b5684105-31e5-42d6-92c9-f3c1da86b741)


2. It is important to open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group. Requests are made via this port

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/15f514c1-aa62-4fb8-9e1f-cc79c78c36ec)



3. ## Installing Packages:

4. 
Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:


#Install apache2


`sudo apt update -y`

`sudo apt install apache2 -y`

`sudo apt-get install libxml2-dev -y`

#Enable following modules:


`sudo a2enmod rewrite`

`sudo a2enmod proxy`

`sudo a2enmod proxy_balancer``

`sudo a2enmod proxy_http`

`sudo a2enmod headers`

`sudo a2enmod lbmethod_bytraffic`


#Restart apache2 service


`sudo systemctl restart apache2`

`sudo systemctl status apache2`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/87e922a2-8451-4abb-b5a9-459ec27cfffd)


Apache2 is running.

## Configure load balancing

Edit the `default.conf` file to add the backend web servers into the loadbalancers proxy for routing.

`sudo vi /etc/apache2/sites-available/000-default.conf`


#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
            
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

Note: Only 2 servers were added to the proxy list and also other ways to route traffic aside `bytraffic` includes `byrequests`, `bybusyness`, `heartbeats` which can be specified in ProxySet lbmethod=? .

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/fef99546-ff15-4751-b028-903ed18d1f31)


Restart the apache2 server `sudo systemctl restart apache2`


4. Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:

`http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/22785e80-28b1-412b-9432-ce3f3d7207e0)



**Note:** If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

`sudo umount -f /var/log/httpd`

To confirm that traffic is routed evenly to both web servers as the load balancer server is receiving traffic (which in our case is by refreshing the webpage) we can check the logs both servers receive 

`sudo tail -f /var/log/httpd/access_log`

Server 1

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/502135a6-0f0f-4707-beba-21f1c66212c8)


Server 2

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/c47f69a4-42c4-487b-ab92-20f29924fd2b)


## Configuring DNS Names (Locally)

In order not to always provide webserver private ip address whenever a new web server needs to be added on the list of loadbalancer proxy, we can specify them on the hosts file and provide a domain name for each which is fine with us. 

 The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB

 #Open this file on your LB server

`sudo vi /etc/hosts`

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

`<WebServer1-Private-IP-Address> Web1`

<`WebServer2-Private-IP-Address> Web2`


 ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/190bef19-0660-41a6-889f-41a7ddf57d8f)


Now you can update your LB config file with those names instead of IP addresses.


`BalancerMember http://Web1:80 loadfactor=5 timeout=1`

`BalancerMember http://Web2:80 loadfactor=5 timeout=1`


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/68d9f1bc-439a-4dab-9e0c-cd1d3a0f076a)


To verify, You can try to curl your Web Servers from LB locally `curl http://Web1` or `curl http://Web2`


curl http://Web1

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/0f350aa9-98f2-41b2-9809-e7250c88cb00)

curl http://Web2

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/19a8c9fc-ec5b-4c13-8477-65153db1d0cb)



## Targed Architecture

Now your set up looks like this:


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/dddf975b-a128-46ba-90f1-e3e3bd29d3e2)

