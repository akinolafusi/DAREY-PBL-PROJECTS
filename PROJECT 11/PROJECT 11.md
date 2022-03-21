#
ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10
#

##
INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE
##

- Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
-  created a new repository and named it ansible-config-mgt.
- Instal Ansible
```
sudo apt update

sudo apt install ansible
```
- Verify Ansible is installed
```
ansible --version
```

![] (https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/d396c302ab0a64c413bd5ec83fc5714f9d09a9f3/PROJECT%2011/ansible%20version.PNG)

- Configure Jenkins build job to save your repository content every time you change it

- Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/d396c302ab0a64c413bd5ec83fc5714f9d09a9f3/PROJECT%2011/freestyle.PNG)

- Configure Webhook in GitHub and set webhook to trigger ansible build

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/d396c302ab0a64c413bd5ec83fc5714f9d09a9f3/PROJECT%2011/webhook.PNG)

- Configure a Post-build job to save all (**) files

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/d396c302ab0a64c413bd5ec83fc5714f9d09a9f3/PROJECT%2011/postbuild.PNG)

-  Test  setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder


```
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/

```
NOTE: ELASTIC IP WAS ASSIGNED TO THE ANSIBLE-JENKINS SERVER TO MAKE SURE THE IP DOESN'T CHANGE IN CASE WE STOP AND START THE SERVER. TO AVOID EDITING THE WEBHOOK OVER AGAIN.

##
Step 2 – Prepare your development environment using Visual Studio Code
##

-  Install Visual Studio

##
Step 3 - BEGIN ANSIBLE DEVELOPMENT
##

-  Create a new branch in respository that will be used for development.

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/d396c302ab0a64c413bd5ec83fc5714f9d09a9f3/PROJECT%2011/branch.PNG)

- Create a directory and name it playbooks – it will be used to store all playbook files.
- Create a directory and name it inventory – it will be used to keep the hosts organised.
- Within the playbooks folder, create the first playbook, and name it common.yml
- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

##
STEP 4 - SETUP AN ANSIBLE INVENTORY FILE
##

- The inventory file will be saved in /inventory/dev folder.
- Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – ssh-agen can be used for this. The key needs to be inported into the ssh-agent:
```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```
- Confirm the key has been added with the command below.
```
ssh-add -l
```
- Now, ssh into your Jenkins-Ansible server using ssh-agent
```
ssh -A ubuntu@public-ip
```
- Update the inventory/dev.yml file with this snippet of code:
```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

![]
##
Step 5 – Create a Common Playbook
##

- Update the playbooks/common.yml file with following code:
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

[image]

##
Step 6 – Update GIT with the latest code
##

- Raise a Pull Request (PR) on github, get the branch peer reviewed and merged to the master branch.
- Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.

##
Step 7 – Run first Ansible test
##

- Execute ansible-playbook command and verify if your playbook actually works:
```
ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml
```

- Optional step – Repeat once again
- Update the ansible playbook with some new Ansible tasks and go through the full checkout -> change codes -> commit -> PR -> merge -> build -> ansible-playbook