# CLIENT/SERVER ARCHITECTURE USING A MYSQL RELATIONAL DATABASE MANAGEMENT SYSTEM

## Understanding Client-Server Architecture
Client-Server Architecture refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another.

In their communication, each machine has its own role: the machine sending requests is usually referred as “Client” and the machine responding (serving) is called “Server”.

A simple diagram of Web Client-Server architecture is presented below:


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/8e89c476-7cac-401e-9ef8-e3eb7ed37c83)

From the example above, a machine that is trying to access a Web site using Web browser or simply ‘curl’ command is a CLIENT, and it sends HTTP requests to a Web server (Apache, Nginx, IIS or any other) over the Internet.

If this concept is extended further and we add a Database Server to our architecture, we can get this picture below:


![image](https://github.com/rxneyo/Darey.io-PBL/assets/125794122/7bfd3371-152c-4b5b-b838-27f780676a44)


In this case, our Web Server has a role of a “Client” that connects and reads/writes to/from a Database (DB) Server (MySQL, MongoDB, Oracle, SQL Server or any other), and the communication between them happens over a Local Network (it can also be Internet connection, but it is a common practice to place Web Server and DB Server close to each other in local network).


The setup on the diagram above is a typical generic Web Stack architecture that you have already implemented in previous projects (LAMP, LEMP, MEAN, MERN), this architecture can be implemented with many other technologies – various Web and DB servers, from small Single-page applications SPA to large and complex portals.



# IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).


## TASK – Implement a Client Server Architecture using MySQL Database Management System (DBMS).


To demonstrate a basic client-server architecture using MySQL Relational Database Management System (RDBMS), by following the below instructions:


1. Create and configure two Linux-based virtual servers (EC2 instances in AWS) using the tags below;

Server A name - `mysql server`
   
Server B name - `mysql client`


2. On `mysql server` Linux Server install MySQL Server software.


3. On `mysql client` Linux Server install MySQL Client software.


