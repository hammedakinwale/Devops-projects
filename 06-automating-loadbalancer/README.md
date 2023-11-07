# Automating load balancer configuration with shell scripting

Streamline your load balancer configuration with shell scripting and simple CI/CD on jenkins. this project demonstrates how to automate the setup and maintenance of your load balancer using freestyle job, enhancing and reducing manual effort 

## Deploying and configuring the webservers.

All the process we need to deploy our webserver has been codified in the shell script below:

```#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2

```

we followed the below steps to run the script.

**Step 1.** provision an `EC2 instance` running ubuntu 20.04.

**Step 2.** Open port 8000 to allow traffic from anywhere using the security group.

**Step 3.** Connect to the webserver via the terminal using ssh client

**Step 4.** Open a file, paste the script above and close the file using the command below:

`sudo vi install.sh`

![The view](./images/1.png)

**Step 5.** change the file to executable file with the command below:

`sudo chmod +x install.sh`

![The view](./images/2.png)

**Step 6.** Run the shell script using the command below.and also make sure to read the instructions in the shell script to learn how to use it.

`./install.sh PUBLIC_IP`

![The view](./images/5.png)

### Deploying and configuring Nginx load balancer 

all the steps followed in the implementing load balancer with Nginx course has been codified in the below script:

read the instructions in the script carefully to learn how to use the script.

```
#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
    sudo touch /etc/nginx/conf.d/loadbalancer.conf

    sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
    sudo chmod 777 -R /etc/nginx/

    
    echo " upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server  "${firstWebserver}"; # public IP and port for webserser 1
            server "${secondWebserver}"; # public IP and port for webserver 2

            }

           server {
            listen 80;
            server_name "${PUBLIC_IP}";

            location / {
                proxy_pass http://backend_servers;   
            }
    } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx

```

#### Steps to run the shell script

**Step 1:** On your terminal, open a file nginx.sh using the command below:

`sudo vi nginx.sh`

**Step 2:** Copy an paste the script inside the file

**Step 3:** Close the file with the code below:

`type esc the shift + :wqa!`

![The view](./images/3.png)

**Step 4:** make the file executable with the command below:

`sudo chmod +x nginx.sh`

![The view](./images/4.png)

**Step 5:** Run the script with command below:

`./nginx.sh PUBLIC_IP Webserver-1 Webserver-2`

![The view](./images/6.png)

### verify if the set up work on any web browser with the ip adresses

![server1 screenshot](./images/7.png)

![first2 server screenshot](./images/8.png)

![loadbalancer screenshot](./images/9.png)