#
PROJECT 9: CONTINOUS INTEGRATION PIPELINE FOR TOOLING WEBSITE
#

###
INSTALL AND CONFIGURE JENKINS SERVER
###
###
Step 1 – Install Jenkins server
###
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins

- install JDK (since Jenkins is a Java-based application)

```
sudo apt update
sudo apt install default-jdk-headless
```
- Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
- Perform initial Jenkins setup.
- From browser access
```
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
```
![]https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/3543bb45974bde363b83dc6e44e6c5027242caa6/project%209/jenkins%20getting%20dtrsted.PNG

- Retrieve it from your server
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
###
Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
###

- Enable webhooks in my your GitHub repository settings
- Go to Jenkins web console, click "New Item" and create a "Freestyle project"
- Add the URL from the project in Github Repo and paste it as the source code in Jenkins
- save the configuration and use the build button in the Project to start the first  build.

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/e03459192fefd726a83a52ed59b4f638c56f5e26/project%209/build%201.PNG)

- Click "Configure" your job/project and add these two configurations
Configure triggering the job from GitHub webhook

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/e03459192fefd726a83a52ed59b4f638c56f5e26/project%209/build%20trigger.PNG)

- Configure "Post-build Actions" to archive all the files into **

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/e03459192fefd726a83a52ed59b4f638c56f5e26/project%209/artifacts.PNG)

- Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.
A build will be triggered automatically using the webhook configured earlier.

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/e03459192fefd726a83a52ed59b4f638c56f5e26/project%209/Build%202.PNG)

- By default, the artifacts are stored on Jenkins server locally
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
#
CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH
#
### 
Step 3 – Configure Jenkins to copy files to NFS server via SSH
###

- Install "Publish Over SSH" plugin.
- Configure the job/project to copy artifacts over to NFS server. 
Navigate to Manage jenkings >>> Configuration system >>> Publish over ssh
```
- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
- Arbitrary name
- Hostname – can be private IP address of your NFS server
- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
```
- Test the configuration and make sure the connection returns Success

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/e03459192fefd726a83a52ed59b4f638c56f5e26/project%209/test-success.jpg)

- Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:
```
SSH: Transferred 25 file(s)
Finished: SUCCESS
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/82f045588f03715f7e989bbd04619f8b6c7ad421/project%209/success.PNG)

