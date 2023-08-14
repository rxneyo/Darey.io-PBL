# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

## Introduction

By now you have learned what Load Balancing is used for and have configured an LB solution using Apache, but a DevOps engineer must be a versatile professional and know different alternative solutions for the same problem. That is why, in this project we will configure an Nginx Load Balancer solution. Aside implementing the load balaning solution with apache.

It is also very important to ensure that connections to your Web solutions are secure and information is encrypted in transit – we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.


When data is moving between a client (browser) and a Web Server over the Internet – it passes through multiple network devices and, if the data is not encrypted, it can be relatively prone to interception by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.

This threat is real – users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.


`SSL and its newer version, TSL` – is a security technology that protects connection from MITM attacks by creating an encrypted session between client (browser) and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.



In this project we will register a website with LetsEnrcypt Certificate Authority, to automate certificate issuance you shall use a shell client recommended by LetsEncrypt – cetrbot.



## Task

This project consists of two parts:

1. Configure Nginx as a Load Balancer

2.Register a new domain name and configure secured connection using SSL/TLS certificates


Our target architecture will look like this:

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/b5a7e164-569e-41b9-a420-13e03705b123)


## CONFIGURE NGINX AS A LOAD BALANCER


To confiure Nginx as a load balancer, one can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx.


1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)

   ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/7bb84bc9-5082-4c3a-bf4b-16f3a6b9ab0b)

 
2.Update `/etc/hosts` file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

Update /etc/hosts file for local DNS with Web Servers names (e.g. Web1 and Web2) and their local IP addresses just like it was done with the apache load balancer.

`sudo vi /etc/hosts`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/77addcec-ba28-4536-9d8c-c866adc64dae)



3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Next is to update the instance and Install Nginx

  `sudo apt update`
  
  `sudo apt install nginx`

  Run 

  `sudo systemctl enable nginx`

  `sudo systemctl start nginx`


  Then ensure Nginx is active and running by 


  `sudo systemctl status nginx`


  ![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/64309e17-8af9-4699-980d-8f5e0bd63f11)


  
Configure Nginx LB using Web Servers’ names defined in `/etc/hosts`


Open the default nginx configuration file

`sudo vi /etc/nginx/nginx.conf`

#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/1d975250-29f4-45ab-90c1-c560d4ae2192)


Restart Nginx and make sure the service is up and running

`sudo systemctl restart nginx`

`sudo systemctl status nginx`

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/805859d5-d564-4ebd-bdd7-4d450fc1303f)



## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

1. Register a new domain name with any registrar

2. Assign an Elastic IP to Nginx LB server and associated domain name with the Elastic IP.

Then open the Amazon EC2 console and select Elastic IPs.

Select the Elastic IP address to associate and choose Actions, Associate Elastic IP address.

For Resource type, choose Instance.

For instance, choose the instance with which to associate the Elastic IP address. You can also enter text to search for a specific instance.

(Optional) For Private IP address, specify a private IP address with which to associate the Elastic IP address.

3. Update A record in your registrar to point to Nginx LB using Elastic IP address

4. Configure Nginx to recognize the new domain name. This was done by Updating the /etc/nginx/nginx.conf file with


`server_name www.<your-domain-name.com>`

In this case, my domain name is toolingneyo.site

![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/172c65ab-443c-4441-8b0b-29252d83817f)


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/d62c330a-88ff-4d3f-b1c1-80895c95ba42)


5. Install certbot and request for an SSL/TLS certificate for the domain name.
  
 N.B: Make sure snapd is active and running on the server.

`sudo systemctl status snapd`


![image](https://github.com/rxneyo/DevOps_Projects/assets/125794122/ec615d9d-b946-493f-a87d-eb7de1c4d067)


Install certbot

`sudo snap install --classic certbot`

6. Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).

   Follow the prompts and instructions displayed


   Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>

You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for your website.

7. Set up periodical renewal of your SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode

`sudo certbot renew --dry-run`

Best practice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:

`crontab -e`

Add the following line to the crontab file

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

Save the crontab file
