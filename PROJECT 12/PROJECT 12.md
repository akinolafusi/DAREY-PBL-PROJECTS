#
 Project 12: Ansible Refactoring, Assignments & Imports
 #

##
Step 1 – Jenkins job enhancement
##
-  Enhance the Jenkins Job by introducing a new plugin - Copy Artifact Plugin

- Go to Jenkins-Ansible server and create a new directory called ansible-config-artifact -  All the artifact from the builds will be stored here
```
sudo mkdir /home/ubuntu/ansible-config-artifact
```
- Change permissions to this directory, so Jenkins could save files there
```
chmod -R 0777 /home/ubuntu/ansible-config-artifact
```
- Install Copy Artifact plugin on Jenkins plugin management console

- Create a New freestyle project with the name save_artifact. The project will be triggered after the completion of Ansible freestle project.

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/b53aeb5b543b23822db20f75b658307b6af2a408/PROJECT%2012/saveart1.PNG)

- The saved artifacts will be sent to /home/ubuntu/ansible-config-artifact which is achieved by creating a build step and choose copy artifacts from other project.

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/b53aeb5b543b23822db20f75b658307b6af2a408/PROJECT%2012/saveart2.PNG)

- Test by making a change in the code on Github. The saved artifact will be seen in 
/home/ubuntu/ansible-config-artifact

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/b53aeb5b543b23822db20f75b658307b6af2a408/PROJECT%2012/savedartinubuntu.PNG)

- ##
REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML
#

Step 2 – Refactor Ansible code by importing other playbooks into site.yml

- Create a new folder called static-assignments.
- Move common.yml file into the newly created static-assignments folder.

- Inside site.yml file, import common.yml playbook.
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
- Run the site.yml playbook against the dev enviroment.
- Create another playbook with the name common-del.yml with the purpose of deletig all the configurations made by common.yml
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```
- update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:

```
sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/dev.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml
```
##
CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’
#

- Step 3 – Configure UAT Webservers with a role ‘Webserver’

- Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.

- To create a role, must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.

- Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)
```
mkdir roles
cd roles
ansible-galaxy init webserver
```
- Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of the 2 UAT Web servers
```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/b53aeb5b543b23822db20f75b658307b6af2a408/PROJECT%2012/uat%20servers.PNG)

- In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path    = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.

![]

- Add more logic to the webserver role. Inside the tasks director of the role, add some configuration into main.yml
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

##
REFERENCE WEBSERVER ROLE
##

- Step 4 – Reference ‘Webserver’ role
Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role.
```
---
- hosts: uat-webservers
  roles:
     - webserver
```
- Refer the uat-webservers.yml role inside site.yml.

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```
##
Step 5 – Commit & Test
##
- Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

- Run the playbook against the uat Inventory
```
sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml
```
[image]
