#
PROJECT 10: Load Balancer Solution With Nginx and SSL/TLS
#
##
CONFIGURE NGINX AS A LOAD BALANCE

- Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
 
 - Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
 
 ![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/48b5c915b4461ce749af32ca48dcad5e0823b8e2/PROJECT%2010/hosts.PNG)

 - Update the instance and Install Nginx

 - Open the default nginx configuration file and paste
 ```
 #insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }
```
- Restart Nginx and make sure the service is up and running
```
sudo systemctl restart nginx
sudo systemctl status nginx
```
##
REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
##

- Register a new domain name with any registrar of your choice in any domain zone (devopsmaestro.com)
- Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP
- Update A record in your registrar to point to Nginx LB using Elastic IP address

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/48b5c915b4461ce749af32ca48dcad5e0823b8e2/PROJECT%2010/DNS.PNG)

- Configure Nginx to recognize your new domain name
Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

- Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running
```
sudo systemctl status snapd
```
- Install certbot
```
sudo snap install --classic certbot
```
- Request your certificate (just follow the certbot instructions
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/48b5c915b4461ce749af32ca48dcad5e0823b8e2/PROJECT%2010/cert.PNG)

- Test secured access to your Web Solution by trying to reach https://devopsmaestro.com

![](https://github.com/akinolafusi/DAREY-PBL-PROJECTS/blob/48b5c915b4461ce749af32ca48dcad5e0823b8e2/PROJECT%2010/cert2.PNG)

- Set up periodical renewal of your SSL/TLS certificate

You can test renewal command in dry-run mode

```
sudo certbot renew --dry-run
```
- Best pracice is to have a scheduled job that to run renew command periodically. Configure a cronjob to run the command twice a day.

- edit the crontab file
```
crontab -e
```
paste
```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```
