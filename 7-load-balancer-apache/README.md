# IMPLEMENTATION OF LOAD BALANCERS WITH NGNIX

## Introduction to load Balancer and Nginx

A load balancer distributes workloads across multiple computers, such as virtual servers. Using a load balancer increases the availability and fault tolerance of your applications.

You can add and remove computers from your load balancer as your needs change, without disrupting the overall flow of requests to your applications.

The load balancer stands in front of the webserver and all traffic gets into it first, it then distribute the traffic evenly across the set of webservers. This ensures no webserver gets overworked consequently improving performance.

Nginx is a versatile software it acts like a webserver reverse proxy, and a load balancer etc.All that is needed is to configure it properly to server to your server.

## Setting Up A Basic Load Balancer.

we will provision two `EC2 instances` running on `ubuntu 22.04`, and also install `NGINX toact` as a load balancer distriuting traffic across web server.

**STEP 1.** Provisioning EC2 instances.

+ Open Aws management Console and click on EC2 instances to launch

![1](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/1eb1230e-01ad-4eca-8294-43a9c099a7a4)

+ Under name provide a unique name for each of your Webserver. In this case `Loadbalancer1` and `Loadbalancer2`

![it should look like this](./images/2.png)

![it should look like this](./images/3.png)

+ Under Application and Os images,click on quick start and Ubuntu 22.4

![it should look like this](./images/4.png)

Under key pair, click on create new key and use it for all the instances you provision. In this case we will use an existing key called `server`.

![it should look like this](./images/5.png)

Finally, click on Launch Instances.

![it should look like this](./images/6.png)

**STEP 2.** Open Port 8000, we will be running our webservers on port 8000 while the load balancers run port 80. we will need to open port 8000 to allow traffic from anywhere. To do this we need to add a rule to the security group of each of our webserver.

+ Click On the instance ID to get of your EC2 instances.

![Loadbalancer1](./images/7.png)

![Loadbalancer2](./images/8.png)

ON the same page scroll down and click on Security

![it should look like this](./images/9.png)

+ Click on edit inbound rule.


![it should look like this](./images/11.png)

Set your rules;

![it should look like this](./images/11.png)

**STEP 3.** After provisioning both servers and openeing the necessary port,Install apache servers on both servers.

To do this we need first of all connect to the servers via ssh to be able to run the installation command on the Terminals

+ To connect the instance, click the instance Id and Click connect.

![Loadbalancer1](./images/12.png)

![Loadbalancer2](./images/13.png)

Copy the shh client and cd into download in your Terminal then paste the ssh client and hit enter

![Loadbalancer1](./images/14.png)
![Loadbalancer2](./images/15.png)

To install Apache server Run `sudo apt update -y`, `sudo apt upgrade -y` and `sudo apt install apache2`

![Loadbalancer1](./images/17.png)
![Loadbalancer2](./images/16.png)

+ To verify if Apache is running on the system we will run `sudo systemctl status apache2`

![Loadbalancer1](./images/18.png)

![Loadbalancer2](./images/19.png)

**STEP 4** Configuring Apache webservers to serve content on port 8000 instead of the default port which is port 80. Then we will create new index.html file. The file contain code to display the public Ip address of the Ec2 instances . We will then override apache webserver default html file with the new file

+ To configure Apache to serve content on port `8000` we will add another listening directive to port `8000`.

we will Run `sudo vi /etc/apache2/ports.conf ` and use text editor to open /etc/apache2/ports.config

![Loadbalancer1](./images/20.png)

![Loadbalancer2](./images/21.png)

+ To open the file /etc/apache2/sites-available/000-default.conf and change port;80 on the virtual host to port; 8000. we will run `sudo vi /etc/apache2/sites-available/000-default.conf`

![Loadbalancer1](./images/22.png)

![Loadbalancer2](./images/23.png)

+ to load the new configurations we will restart apache with the command below:

`sudo systemctl restart apache2`

![Loadbalancer1](./images/24.png)

![Loadbalancer2](./images/25.png)

+ To open a new index.html file

+ the Run `sudo vi index.html`

+ and paste the code below and get the public Ip address to replace the placeholder text for Ip address in the html file.

```        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>

```

![Loadbalancer1](./images/26.png)

![Loadbalancer2](./images/27.png)

Change the file ownership of index.html with `sudo chown www-data:www-data ./index.html`

![Loadbalancer1](./images/29.png)

![Loadbalancer2](./images/30.png)

**Overriding the default HTML file of the apache webserver**

+ we will replace the default html file with our new html with `sudo cp -f ./index.html /var/www/html/index.html`

![Loadbalancer1](./images/31.png)

![Loadbalancer2](./images/32.png)

+ to load the new configurations we will restart apache with the command below:

`sudo systemctl restart apache2`

![Loadbalancer1](./images/33.png)

![Loadbalancer2](./images/34.png)

**STEP 5.** CONFIGURING NGINX AS LOAD BALANCER Prerequisite:

+ provision a new EC2 Instance running Ubuntu 22.4
+ make sure port 80 is opened to accept traffic from anywhere.
+ SSH into the instance via the Terminal

**Then INSTALL NGINX**

+ To Install Nginx Run `sudo apt update -y && sudo apt install nginx -y`

![it should look like this](./images/35.png)

+ Verify that Nginx is installed successfully with `sudo systemctl status nginx`

![it should look like this](./images/36.png)

+ Open a configuration file where you are going to paste the code for nginx to act as a load-balancer, with `sudo vi /etc/nginx/conf.d/loadbalancer.conf`

Then paste 
``` upstream backend_servers {

     # your are to replace the public IP and Port to that of your webservers
     server 127.0.0.1:8000; # public IP and port for webserser 1
     server 127.0.0.1:8000; # public IP and port for webserver 2

 }

 server {
     listen 80;
     server_name <your load balancer's public IP addres>; # provide your load balancers public IP address

     location / {
         proxy_pass http://backend_servers;
         proxy_set_header Host $host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     }
 }

```

![it should look like this](./images/37.png)

+ Upstream backend_servers : Define a group of backend servers
+ The server lines inside the upstream block list the addresses and the ports of your backend-servers.
+ Proxy_pass inside the location block sets up the load balancing, passing the request to the backend servers.
+ The proxy_set_header lines pass necessary header to the backend servers to correctly handled the request

test the configuration with the command below:

`sudo nginx -t`

![it should look like this](./images/38.png)

+ to load the new configurations we will restart apache with the command below:

`sudo systemctl restart nginx`

![it should look like this](./images/39.png)

+ paste the Ip-address of the Nginx  In any web browser and hit enter.

![it should look like this](./images/40.png)
