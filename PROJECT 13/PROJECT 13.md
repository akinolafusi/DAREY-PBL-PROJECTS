#
Introducing Dynamic Assignment Into Our structure
#

- In the Github repository, create a new branch called Dynamic-assignments.
- Create a new folder by the name dynamic-assignments.
-  create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.
- Now paste the instruction below into the env-vars.yml file
```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

[image]

##
UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS
##

- Update site.yml file to make use of the dynamic assignment
```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```

[image]

# 
Download Mysql Ansible Role, We will be using a MySQL role developed by geerlingguy.

- On ansible-config-mgt run
```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```
- Inside roles directory create the new MySQL role with

[image]
```
ansible-galaxy install geerlingguy.mysql
```
rename the folder to mysql
```
mv geerlingguy.mysql/ mysql
```
- Now it is time to upload the changes into my GitHub

```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
- Create a Pull Request and merge it to main branch on GitHub.

[image]

#
LOAD BALANCER ROLES

- We want to be able to choose between two load balancers which one to use for a particular enviroment. Beween NGINX and APACHE

- Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively and set both values to false

[image]

- Declare another variable in both roles load_balancer_is_required and set its value to false as well

[image]

- Update both assignment and site.yml files respectively
loadbalancers.yml
```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

site.yml file

```
- name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 
```