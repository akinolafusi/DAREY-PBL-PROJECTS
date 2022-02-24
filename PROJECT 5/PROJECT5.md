#
IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)

- Create and configure two Linux-based virtual servers (EC2 instances in AWS).
```
Server A name - `mysql server`
Server B name - `mysql client`
```
- On mysql server Linux Server install MySQL Server software
```
sudo apt install mysql-server
```
- On mysql client Linux Server install MySQL Client software
```
sudo apt install mysql
```
- open port 3306 for mysql but only allow the mysql client IP for security purposes.
[image]

- configure MySQL server to allow connections from remote hosts
```
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
[image]
- create a database and user on the server
```
mysql> create database;
mysql> CREATE USER 'myuser'@'remote_server_ip' IDENTIFIED BY 'mypass';

mysql> GRANT ALL PRIVILEGES ON 'db' TO 'newuser'@'localhost';
mysql> FLUSH PRIVILEDGES;
mysql> exit
```
- connect into the mysql server using mysql utility and not ssh
```
sudo mysql -u <user> -h <server private ip> -p
```
