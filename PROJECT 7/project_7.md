## DEVOPS TOOLING WEBSITE SOLUTION (project 7)

## Prepare NFS Server
- Spin up REHEL 8 on AWS and configure LVM on the Sever
- Create 3 volumes same as the server
- Following the steps of creating and mounting logical volumes in PROJECT 6 lv-apps, lv-logs and lv-opt is created.

- The Volumes should so be formated into xfs
```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/logs-opt
```
- lv-apps is mounted to /mnt/apps (for webservers)
```
sudo mount /dev/webdata-vg/lv-apps /mnt/apps/
```
- lv-logs is mounted to /mnt/logs (for webserver logs)
```
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
```

-lv-opt is mounted to /mnt/opt (for jenkins in next project)
```
sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```
- Update the /etc/fstab to make the mount persist after restarting the server.

The UUID of the device will be used to update the /etc/fstab file
```
sudo blkid
```
[image]
```
sudo vi /etc/fstab
```
[image]

- Test the configuration
```
sudo mount -a
```
- Reload the daemon
```
sudo systemctl daemon-reload
```
Verify the setup
```
df -h
```
[image]

### Install NFS server, configure it to start on reboot and make sure it is u and running ###

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
- Export the mounts for webservers’ subnet cidr to connect as clients. 

- Set up permission that will allow our Web servers to read, write and execute files on NFS
```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```

- Configure access to NFS for clients within the same subnet (172.31.80.0/20)
```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)


sudo exportfs -arv
```
- Check which port is used by NFS and open it using Security Groups
```
rpcinfo -p | grep nfs
```
- In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049
 
## CONFIGURE THE DATABASE SERVER

