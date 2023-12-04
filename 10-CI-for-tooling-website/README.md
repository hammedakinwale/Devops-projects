# Tooling Website deployment automation with Continuous Integration. Introduction to Jenkins

![the architecture](./images/1.PNG)

**INSTALL AND CONFIGURE JENKINS SERVER**

**Step 1** – Install Jenkins server

1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

![it should look like this](./images/2.PNG)

2. Install Jenkins

```
#!/bin/bash
# jenkins installation script
# update the server repository
sudo apt update -y
# This is the Debian package repository of Jenkins to automate installation and upgrade. To use this repository, first add the key to your system:
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
# Then add a Jenkins apt repository entry:
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
# Update your local package index, then finally install Jenkins:
sudo apt-get update -y
sudo apt-get install fontconfig openjdk-11-jre -y
sudo apt-get install jenkins -y
echo "Jenkins installation successfull"
# Unlock Jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

3. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group

![it should look like this](./images/3.PNG)

4. Perform initial Jenkins setup.

From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

You will be prompted to provide a default admin password

to unluck jenkins cat this `/var/lib/jenkins/secrets/initialAdminPassword` in your command line to et password

![it should look like this](./images/4.PNG)

Then you will be asked which plugings to install - choose suggested plugins.

![it should look like this](./images/5.PNG)

you will be asked to create new admin user

![it should look like this](./images/6.PNG)

![it should look like this](./images/7.PNG)

![the dashboard](./images/7.PNG)

**Step 2** - Configure Jenkins to retrieve source codes from GitHub using Webhooks 

In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a 'build' task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings

![the dashboard](./images/9.PNG)

2. Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![the dashboard](./images/10.PNG)

To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

![the dashboard](./images/11.PNG)

In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![the dashboard](./images/12.PNG)

Save the configuration and let us try to run the build. For now we can only do it manually.
Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1

![the dashboard](./images/13.PNG)

You can open the build and check in "Console Output" if it has run successfully.

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

3. Click "Configure" your job/project and add these two configurations

Configure triggering the job from GitHub webhook:

![the dashboard](./images/15.PNG)

![the dashboard](./images/14.PNG)

You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as 'push' because the changes are being 'pushed' and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
`

**Step 3** - Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.
Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called `Publish Over SSH`.

1. Install "Publish Over SSH" plugin.

![the dashboard](./images/16.PNG)

![the dashboard](./images/17.PNG)

On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.
On "Available" tab search for "Publish Over SSH" plugin and install it

2. Configure the job/project to copy artifacts over to NFS server.

On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.
Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server.

1. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
2. Arbitrary name
3. Hostname - can be private IP address of your NFS server
4. Username - ec2-user (since NFS server is based on EC2 with RHEL 8)
5. Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS

![the dashboard](./images/18.PNG)

Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"

```
SSH: Transferred 25 file(s)
Finished: SUCCESS
```

![the dashboard](./images/19.PNG)

To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file

`cat /mnt/apps/README.md`

If you see the changes you had previously made in your GitHub – the job works as expected.

If you are having issues with the build failing or been unstable, follow the steps below to correct such.

+ In your NFS server change permissions for the /mnt/apps directory

```
sudo chmod 777 /mnt/apps
sudo chown nobody:nobody /mnt/apps
sudo chmod -R 777 /mnt
sudo chown -R nobody:nobody /mnt
```