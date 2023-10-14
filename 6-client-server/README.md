# UNDERSTANDING CLIENT-SERVER ARCHITECTURE

## Client Server Architecture With MySQL

### What is client-server architecture?

Client-server refers to an Architecture in which two or more computers are connected over a network to send and receive request s between each other.

In their communication, each machine has its own role The machine sending the request is referred to as the **CLIENT** and the machine responding(serving) is referred to as the **SERVER**.

## IMPLEMENTING CLIENT-SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)

### To demonstrate basic client-server using MySQL RDBMS

**To create and configure two linux based Virtual servers(EC2 instances on AWS)**

**1.** create two servers:

+ Server A name `mysql-server` for the serving machine
+ Server B name `mysql-client` for the requesting machine

![it should look like this](./images/ins.png)

**2.** Connect to the `mysql-server` 

![it should look like this](./images/sr.png)

**3.** And Install` mysql-client` software with the command below:

`sudo apt install mysql-client`

![it should look like this](./images/sql.png)

Connect to the `mysql-client` 

![it should look like this](./images/cl.png)

And Install MySQL software with the command below:

`sudo apt install mysql-server`

+ install Mysql for the client server with `sudo apt install mysql-client`

![it should look like this](./images/lqs.png)

**4.** By default both EC2 virtual servers are located in the same local virtual network. for to be able tocommunicate with each other using the local **IP ADRRESS** 

To use **mysql-server** local Ip address to connect from the client

Change the inbound rule to TCP to port 3306 and connect to Clients **Private Ip address**


![it should look like this](./images/da.png)

**5.** To connect Client to Server We need to configure MySQL-Server to allow connection from remote host with the command below.

` sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf ` and change the value of nind-address to `0.0.0.0`

![it should look like this](./images/conf.png)

**6.** To Run connect the client-mysql to the Server-mysql without using SSH

Run the code below to create a user called Client and the user should be identified by a password PassWord.2 on the MySQL-Server

Run the code below to create a user called Client and the user should be identified by a password `PassWord.2` on the `MySQL-Server`.

log in with the command belwo:

`sudo mysql`

![it should look like this](./images/log.png)

paste this `DATAS USER 'Datas_user'@'private-Ip' IDENTIFIED BY 'PassWord.2';
` in mysql console and run.


![it should look like this](./images/jh.png)

then grant permision to `Data-user` with the below command.

`GRANT ALL ON Datas_database.* TO 'Clientes_user'@'private-Ip';
FLUSH PRIVILEGES;`


![it should look like this](./images/cli.png)

**7.** Then From the client server run `mysql -u Datas_user -p -h <private-ip-address> `using the private Ip of the *Server

