# WEB STACK IMPLEMENTATION [LEMP STACK]

**HINT** : in previuos project we used putty on windows to connect to our EC2 instance, but there is a simpler way that do not require conversion of .pem Key to .ppk using git bash and run the following commen:

`ssh -i <Your-Private-Key.pem> ubuntu@<EC2-public-IP-address>`

![and it should look like this.](./images/conf.png)


## STEP 1 - INSTALLING THE NGINX SERVER

To install nginx, we use the apt package manager by typing the command below:

`sudo apt install nginx`

![and it should look like this.](./images/nginx.png)

To confirm that nginx was successfully installed and running as a service in Ubuntu, run the command below:

`sudo systemctl status nginx`

![and it should look like this.](./images/run.png)

Now we will check if we can access it locally through our Ubuntu shell using the coomand below:

first of all go to aws console to add http rule:

`$ curl http://localhost:80 or $ curl http://127.0.0.1:80`

![and it should look like this.](./images/test.png)

you can also check your ip address in ubuntu instead of going to aws console with the below command:

`curl -s http://169.254.169.254/latest/meta-data/public-ipv4`


then we go to the browser to check confirm if nginx is successfully installed with the below **URL**

`http://<Public-IP-Address>:80`

![and it should look like this.](./images/http.png)

the above webpage will be display if the web server is inatall successfully.

## STEP 2 - INSTALLING MySQL.

To install mysql, we use the apt package manager by typing the command below:

`sudo apt install mysql-server`

![and it should look like this.](./images/mysql.png)

When mysql installation is finished, log in to the mysql console with the command below:

`sudo mysql`

![and it should look like this.](./images/log.png)

Then we run a security script that removes insecure default settings and we will set a password for the root user before running the script. To set the password for the root user, we run the scroipt below:

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

![and it should look like this.](./images/pass.png)

Then we exit the MySQL shell with the command below:

`exit`

![and it should look like this.](./images/exit.png)

After that, then we run the interactive script with the command below:

`sudo mysql_secure_installation`

![and it should look like this.](./images/pass.png)

![and this.](./images/pass2.png)

When finished then we test if we are able to log in to the MySQL console with the command below:

`sudo mysql -p`

![and it should look like this.](./images/in.png)

exit with the command below:

`exit`

![and it should look like this.](./images/exit2.png)

## STEP 3 - INSTALLING PHP

We will install php-fpm and php-mysql by using the apt package manager by running the command below:

`sudo apt install php-fpm php-mysql`

![and it should look like this.](./images/php.png)

![and this.](./images/php2.png)

![and this.](./images/php3.png)

## STEP 4 - CONFIGURING NGINX TO USE PHP PROCESSOR

Before we start, we will create the root web directory for our_domain with the command below:

`sudo mkdir /var/www/projectLEMP`

![and it should look like this.](./images/dir.png)

Next, we will asign the ownership of the directory by running:

`sudo chown -R $USER:$USER /var/www/projectLEMP`

![and it should look like this.](./images/di.png)

Then we open a new configuration file in Nginx's `sites-available` directory by running the command below:

`sudo nano /etc/nginx/sites-available/projectLEMP`

And pasting pasting in the below bare-bones configurations:

```
#/etc/nginx/sites-available/projectLEMP

server {
        listen 80; server_name projectLEMP <www.projectLEMP>;
        root /var/www/projectLEMP;

        index index.html index.htm index.php;

        location / {
        try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        }
        location ~ /\.ht {
                deny all;
        }
}

```


![and it should look like this.](./images/vim.png)

After that wewill activate our configuration by linking to the config file from Nginx's sites-enabled directory by running the command below:

`sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`

![and it should look like this.](./images/confirm.png)

Then we test our configuration for any syntax errors with the command below:

`sudo nginx -t `

![and it should look like this.](./images/ok.png)

Then we also need to disable default Nginx host that is currently configured to listen on port 80, for this run the command below:

`sudo unlink /etc/nginx/sites-enabled/default`

![and it should look like this.](./images/kill.png)

We will reload NGINX to apply these changes and then check that everything is working correctly with the command below:

`sudo systemctl reload nginx`

![and it should look like this.](./images/correct.png)

Now we will create an index.html file in the /var/www/projectLEMP directory to test that our new server block works as expected and echo the following scripts in it:

`sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html`

![and it should look like this.](./images/ht.png)

and then we will go to the browser and open our website URL using its IP address with the below **URL** :

`http://<Public-IP-Address>:80`


![and this will texts display.](./images/dis.png)

## STEP 5 - TESTING PHP WITH NGINX

at this point our lemp stack is completely installed and fully operational.
we can test it to validate that Nginx can correctly han.php of to our php processor.

We can do this by creating a test PHP file in document root by running the script below:

`nano /var/www/projectLEMP/info.php`

Add the following scripts into it:

`<?php phpinfo();`

![and it should look like this.](./images/nan.png)

Then we can access the page in the browser by visiting the public address followed by /info.php. Then a web page containing detailed information about the server will be displayed:

![and it should look like this.](./images/ubuntu.png)

After checking the relevant information about the PHP server through that page, it is best to remove the file created as it contains sensitive informations by running the command below:

`sudo rm /var/www/your_domain/info.php`

![and it should look like this.](./images/rm.png)

## STEP 6 - RETRIEVING DATA FROM MySQL DATABASE WITH PHP.

First of all we will connect to the MySQL console using the root account by running the command below:

![and it should look like this.](./images/logb.png)

Then we will create a new database, by running the command below:

`CREATE DATABASEexample_database;`


![and it should look like this.](./images/new.png)

Next, we will create a new user name `example_user` and grant him full privileges on the database just created and password. To do so, run with the command below:

`CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

![and it should look like this.](./images/nu.png)

Now we need to give the newly created user permission over the database by running the command below:

`GRANT ALL ON example_database.* TO 'example_user'@'%';`

![and it should look like this.](./images/ny.png)

Then we will exit with the command below:

`exit`

![and it should look like this.](./images/ex.png)

Then we will test if the new user has proper permissions by using the command below:

`mysql -u example_user -p`

![and it should look like this.](./images/nyp.png)

Now, we'll create a table named todo_list with the command below:

`CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT,content VARCHAR(255),PRIMARY KEY(item_id));`

![and it should look like this.](./images/todo.png)

After creating the table, we might want to add a few rows in the test table by running this command and repeating it with different values:

`INSERT INTO example_database.todo_list (content) VALUES ("My first important item");`

![and it should look like this.](./images/fifth.png)

To confirm that the data was successfully saved to the table we will run the command below:

`SELECT * FROM example_database.todo_list;`

![and it should look like this.](./images/all.png)

After confirming we will exit with:

`exit`

![and it should look like this.](./images/exi.png)

Now we can create a PHP script that will connect to MySQL and query for our content and pasting the below content into the todo_list.php by running the command below:

`nano /var/www/projectLEMP/todo_list.php`

![and it should look like this.](./images/vn.png)

Then we can access the page in our web browser by pasting the public IP address followed by /todo_list.php:

![and it should look like this.](./images/to.png)