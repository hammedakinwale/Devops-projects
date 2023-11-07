# Implementing Wordpress Website With LVM

## **STEP 1.** Preparing Web Server

**1.** Launch an EC2 instance that will server as "Web server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10GB.

![the view](./images/1.png)

+ Attach all volumes to the web server EC2 instance

![the view](./images/3.png)

+ SSH into webserver EC2 instance

![the view](./images/2.png)

+ and view the disks attached to the instance with `lsblk` command.

![the view](./images/4.png)

+ use `df -h` To see all mounts and free spaces on the server

![the view](./images/5.png)

use `gdisk` to create single partitions on each volume on the server.

![for xvdf](./images/6.png)

![for xvdg](./images/7.png)

![for xvdh](./images/8.png)

![for xvdh](./images/9.png)

Install `LVM2` with `sudo yum install lvm2` the package for creating logical volumes on a linux server.

![the view](./images/10.png)

+ use `sudo pvcreate <partition_path>` to to march each of the disks as a physical volume.

![the view](./images/11.png)

+ verify if the physical volume has been created with `sudo pvs`

![the view](./images/13.png)

+ Next we will use vgcreate to add up each of the 3 physical volumes into a volume group name webdata-vg.

![the view](./images/12.png)

use `lvcreate` utility to create two logical volumes with the below commands:

```
sudo lvcreate -n apps-lv -L 14G webdata-vg

sudo lvcreate -n logs-lv -L 14G webdata-vg
```

![the view](./images/14.png)

verify that your logical volume has been created with `sudo lvs`

![the view](./images/15.png)

+ use mkfs.ext4 to format the logical volumes with ext4 filesystem with the command below:

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

![the view](./images/16.png)

+ create /var/www/html with `sudo mkdir -p /var/www/html` to store website files

![the view](./images/17.png)

+ create /home/recovery/logs with `sudo mkdir -p /home/recovery/logs` to store backups of log data

![the view](./images/18.png)

+ Mount /var/www/html on apps-lv logical volume with the below command:

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/
`

![the view](./images/20.png)

use rsync utility to backup all the log directory /var/log into /home/recovery/logs (this is requred before mounting the file system)

![the view](./images/19.png)

+ Mount /var/log on logs-lv logical volume with `sudo mount /dev/webdata-vg/logs-lv /var/log
` (note that all the existing data on /var/log will be deleted)

![the view](./images/21.png)

+ restore log files back into /var/log directory with `sudo rsync -av /home/recovery/logs/log/. /var/log
`

![the view](./images/22.png)

update `/etc/fstab` file with `sudo blkid` so that the mount configuration will persist after restarting the server

![the view](./images/23.png)

update /etc/fstab file with `sudo vi /etc/fstab`

![the view](./images/24.png)

test the configuration and reload the daemon with `sudo mount -a
sudo systemctl and daemon-reload
` respectively

verify your setup by running `df -h`

![the view](./images/25.png)

# installing wordpress and configuring to use mysql data base

**STEP 2.** Preparing The DataBase Server

+ all the steps taken to configure web-server are repeated here. but changed apps-lv logical volume to db-lv

![the view](./images/26.png)

**STEP 3.** Installing wordpress on webserver

+ install the epel-repository with the command below:

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![the view](./images/27.png)

+ and install the remi-repository with the command below:

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

+ check the available php to be install and install them with and commands below:

```
sudo dnf module list php
sudo dnf module reset php
sudo dnf module enable php:remi-7.4
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
```

![the view](./images/28.png)

+ and verify with `php -v `

![the view](./images/29.png)

+ start and enable `php` with:

```
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

![the view](./images/30.png)

+ check `php` status with 

`sudo systemctl status php-fpm`

![the view](./images/31.png)

+ To instruct SELinux to allow Apache to execute the PHP code via PHP-FPM run the command below:

`setsebool -P httpd_execmem 1`

+ install httpd with

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

![the view](./images/32.png)

+ Finally, restart Apache web server for PHP to work with Apache web server. with:

`sudo systemctl restart httpd`

![the view](./images/39.png)

+ then load the public ip on browser

![the view](./images/40.png)

+ install wordpress
    1. makedirectory with `mkdir wordpress` and `cd wordpress`

    2. install wordpress with `sudo wget http://wordpress.org/latest.tar.gz`

    3. extract with `sudo tar xzvf latest.tar.gz`

    3. a new directory is created called `wordpress. cd into `wordpress/`

    3. copy contents of wp-config-sample.php to wp-config.php with `sudo cp -R wp-config-sample.php wp-config.php`

    4. copy wordpress to /var/www/html/

    5. copy the content of wordpress/ to /var/www/html with `sudo cp -R wordpress/. /var/www/html`

**STEP4** Installing MySQL on DB Server EC2 with:

`sudo yum install mysql-server`

![the view](./images/33.png)

to verify that system is up and running we reboot and enable with:

`sudo systemctl start mysqld

sudo systemctl enable mysqld
`

+ check the status with `sudo systemctl status mysqld`

![the view](./images/42.png)

run `sudo mysql_secure_installation` to activate secure installation

**STEP5** Set Up DB Server to work with wordpress

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

![the view](./images/34.png)

+ set the bind address with `sudo vi /etc/my.cnf`

![the view](./images/43.png)

![the view](./images/34.png)

+ Ensured that port 3306 is enabled on our db server to allow our web server to access the database server.

![the view](./images/35.png)

### Connect Web Server to DB Server

on webserver edit /var/www/html with `sudo vi wp-config.php` 

![the view](./images/44.png)

runn ` sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup` to disable apache default page

+ Install mySQl client on the web server to connect to the db server

run `sudo yum install mysql

sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
`
![the view](./images/37.png)

+ then configure bind and configure.php and load the webserver public ip address

![the view](./images/46.png)
![the view](./images/47.png)
![the view](./images/48.png)
![the view](./images/49.png)
