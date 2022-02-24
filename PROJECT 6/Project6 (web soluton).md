##
 PROJECT 6: WEB SOLUTION WITH WORDPRESS
___

###
- Prepare a Web Server on AWS and add 3 volumes in the same AZ as the server.

- Attach all the 3 volumes to the Web server *one by one*

- Head over to the Linux terminal for configuration.

- Check the volumes attached to the server using this commnad
```
lsblk
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/lsblk%20commad%20output.PNG)

- Check all mounts on the server and free space
```
df -h
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/df%20-h.PNG)

- Use gdisk utility to create a single partition on each of the 3 disks

- Install lvm2 package
```
sudo yum install lvm2
```
- check for available partitions
```
sudo lvmdiskscan
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/lvmdiskscan.PNG)

- Mark each of the 3 disks as physical volume
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
- Verify that your Physical volume has been created successfully
```
sudo pvs
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/sudo%20pvs.PNG)

- add all 3 PVs to a volume group (VG). Name the VG webdata-vg
```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
- Verify that VG has been created successfully
```
sudo vgs
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/sudo%20vgs.PNG)

- create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
- Verify the Logical Volume has been created successfully
```
sudo lvs
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/sudo%20lvs.PNG)

- Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/lsblk2.PNG)

- Format the logical volumes with ext4 filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
- Create /var/www/html directory to store website files
```
sudo mkdir -p /var/www/html
```
- Create /home/recovery/logs to store backup of log data
```
sudo mkdir -p /home/recovery/logs
```
- Mount /var/www/html on apps-lv logical volume
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
-  backup all the files in the log directory /var/log into /home/recovery/logs *(This is required before mounting the file system)*
```
sudo rsync -av /var/log/. /home/recovery/logs/
```
- Mount /var/log on logs-lv
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```
- Restore log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/. /var/log
```
- Update /etc/fstab file so that the mount configuration will persist after restart of the server.
#
UPDATE THE `/ETC/FSTAB` FILE

- The UUID of the device will be used to update the /etc/fstab file.
``` 
sudo blkid to get the UUID
sudo vi /etc/fstab
```
- Test the configuration and reload the daemon
```
sudo mount -a
 sudo systemctl daemon-reload
 ```
 - Verify your setup
 ```
 df -h
 ```
 ![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/df%20-h2.PNG)

 #
 Step 2 — Prepare the Database Server
 
 - Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

#
Step 3 — Install WordPress on your Web Server EC2

- Update the repository
```
sudo yum -y update 
```
- Install wget, Apache and it’s dependencies
```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

- Start Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd
```
- install PHP and it’s depemdencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
- Restart Apache

```
sudo systemctl restart httpd
```

- Download wordpress and copy wordpress to var/www/html
```
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
```
- Configure SELinux Policies
```
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
```

#
Step 4 — Install MySQL on your DB Server EC2

```
sudo yum install mysql-server
```

- Verify and enable Mqsql
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
#
Step 5 — Configure DB to work with WordPress

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
#
Step 6 — Configure WordPress to connect to remote database

- Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
- Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/4efee14dd08608bba19af658303e17c283c333c5/PROJECT%206/showdatabases.PNG)

- Change permissions and configuration so Apache could use WordPress:
- Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)
- 
- Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/
![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/b5832af441e284e9c052b3bf9cf58e4333cf0e24/PROJECT%206/success.PNG)

#
SUCCESS



