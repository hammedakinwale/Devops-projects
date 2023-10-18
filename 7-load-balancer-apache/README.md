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

![2](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/d6f1745f-fb82-4030-93be-8e88601bf723)

![3](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/dbc0d11c-ef2c-40eb-8526-69b190676ea8)

+ Under Application and Os images,click on quick start and Ubuntu 22.4

![4](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/e4eda349-5856-48ac-b12d-6e49cedf734a)

Under key pair, click on create new key and use it for all the instances you provision. In this case we will use an existing key called `server`.

![5](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/802a13df-4ad8-47d7-beed-50e52b76711e)

Finally, click on Launch Instances.

![6](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/effe4776-14b5-4500-ad1e-40b96acd4af7)


**STEP 2.** Open Port 8000, we will be running our webservers on port 8000 while the load balancers run port 80. we will need to open port 8000 to allow traffic from anywhere. To do this we need to add a rule to the security group of each of our webserver.

+ Click On the instance ID to get of your EC2 instances.

Loadbalancer1
![7](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/001946bb-33b5-48e0-b708-0231a1c9cae9)

Loadbalancer2
![8](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/6ef978ce-cd05-4a4e-84af-3e7f262201b3)

ON the same page scroll down and click on Security

![9](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/800ad0c4-b187-4d3a-8216-48bfaa1bf88e)

+ Click on edit inbound rule.

![11](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/3bb31c46-dda4-49cf-995d-897a18a8ef28)

Set your rules;
![11](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/3bb31c46-dda4-49cf-995d-897a18a8ef28)

**STEP 3.** After provisioning both servers and openeing the necessary port,Install apache servers on both servers.

To do this we need first of all connect to the servers via ssh to be able to run the installation command on the Terminals

+ To connect the instance, click the instance Id and Click connect.

Loadbalancer1
![12](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/0b12eae9-73a3-46cc-a375-b4e13e00bc18)

Loadbalancer2
![13](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/47976474-8f5a-423e-844f-ffcd62049682)

Copy the shh client and cd into download in your Terminal then paste the ssh client and hit enter

Loadbalancer1
![14](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/ab590a99-a579-4495-9ce0-f9d092c01059)

Loadbalancer2
![15](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/51ca119f-e2f2-482f-bb84-10daa3a16715)

To install Apache server Run `sudo apt update -y`, `sudo apt upgrade -y` and `sudo apt install apache2`

Loadbalancer1
![17](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/d5a1a4ff-5fe5-4a61-bd1e-004ab56eaa70)

Loadbalancer2
![16](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/a565a630-b298-4a2f-8f94-1ec0d3e3107d)

+ To verify if Apache is running on the system we will run `sudo systemctl status apache2`

Loadbalancer1
![18](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/eded0ff7-bfa6-430d-86a2-d4ed305a817b)

Loadbalancer2
![19](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/6afe4020-5022-4ab3-9a92-b7900ab41697)

**STEP 4** Configuring Apache webservers to serve content on port 8000 instead of the default port which is port 80. Then we will create new index.html file. The file contain code to display the public Ip address of the Ec2 instances . We will then override apache webserver default html file with the new file

+ To configure Apache to serve content on port `8000` we will add another listening directive to port `8000`.

we will Run `sudo vi /etc/apache2/ports.conf ` and use text editor to open /etc/apache2/ports.config

Loadbalancer1
![20](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/f2f12009-9bfe-4a27-8e51-9cbfeaa929c4)

Loadbalancer2
![21](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/b4a8ac9a-87aa-4346-b9c1-105e5945035c)

+ To open the file /etc/apache2/sites-available/000-default.conf and change port;80 on the virtual host to port; 8000. we will run `sudo vi /etc/apache2/sites-available/000-default.conf`

Loadbalancer1
![22](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/4ba064d2-155a-43bb-87c4-9a6d7842f769)

Loadbalancer2
![23](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/a1c1280a-0673-47fa-a211-1a3742f39387)

+ to load the new configurations we will restart apache with the command below:

`sudo systemctl restart apache2`

Loadbalancer1
![24](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/f69a26a9-788e-4904-9e82-e5bd61abe0d6)

Loadbalancer2
![25](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/4c077cb7-6a92-4eb6-b9f9-aa43d6361107)


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

Loadbalancer1
![26](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/26ebf859-7ac7-4195-9408-4c15a8a3331a)

Loadbalancer2
![27](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/ab749256-8950-4722-a470-6856ce1bcd4a)

Change the file ownership of index.html with `sudo chown www-data:www-data ./index.html`

Loadbalancer1
![29](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/35fa6edd-7c22-4385-bf8e-1a41f56d340d)

Loadbalancer2
![30](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/53ab97e4-9cdf-421e-9ca0-7fb1dc625f79)

**Overriding the default HTML file of the apache webserver**

+ we will replace the default html file with our new html with `sudo cp -f ./index.html /var/www/html/index.html`

Loadbalancer1
![31](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/10533dcf-40c4-4c76-a7a6-3d715f0deb81)

Loadbalancer2
![32](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/64cce274-d9ad-4d18-a4d1-a509db09c489)

+ to load the new configurations we will restart apache with the command below:

`sudo systemctl restart apache2`

Loadbalancer1
![33](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/ecd37684-371a-4bdc-8c57-8333f49878bd)

Loadbalancer2
![34](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/fe9819fb-e1a7-45b2-8dd8-b3b2aa2c76df)

**STEP 5.** CONFIGURING NGINX AS LOAD BALANCER Prerequisite:

+ provision a new EC2 Instance running Ubuntu 22.4
+ make sure port 80 is opened to accept traffic from anywhere.
+ SSH into the instance via the Terminal

**Then INSTALL NGINX**

+ To Install Nginx Run `sudo apt update -y && sudo apt install nginx -y`

![35](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/5cf35e21-853b-4e75-8a08-86fa19b5fb61)

+ Verify that Nginx is installed successfully with `sudo systemctl status nginx`

![36](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/8db1549a-30e0-41f7-a6da-931ab2a0182a)

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

![37](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/5628bed2-8027-4103-a347-9111065a6d8f)

+ Upstream backend_servers : Define a group of backend servers
+ The server lines inside the upstream block list the addresses and the ports of your backend-servers.
+ Proxy_pass inside the location block sets up the load balancing, passing the request to the backend servers.
+ The proxy_set_header lines pass necessary header to the backend servers to correctly handled the request

test the configuration with the command below:

`sudo nginx -t`

![38](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/35127d1a-a489-46aa-9fdb-e3f7cf1c8b01)

+ to load the new configurations we will restart apache with the command below:

`sudo systemctl restart nginx`

![39](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/80ca23c8-dccd-4bcf-87ef-76cb7e71382f)

+ paste the Ip-address of the Nginx  In any web browser and hit enter.

![40](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/3b52121d-2155-4997-aab7-062faf678405)

