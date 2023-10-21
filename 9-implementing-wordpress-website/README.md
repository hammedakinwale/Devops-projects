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