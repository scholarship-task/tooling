
# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

This project add a load-balancer to the existing architecture so that the user will only be able to access the service using only one IP address. 

NGINX is open source software for web serving, reverse proxying, caching, load balancing, media streaming, and more. It started out as a web server designed for maximum performance and stability.
"Load balancing refers to efficiently distributing incoming network traffic across a group of backend servers, also known as a server farm or server pool.

Modern high‑traffic websites must serve hundreds of thousands, if not millions, of concurrent requests from users or clients and return the correct text, images, video, or application data, all in a fast and reliable manner. To cost‑effectively scale to meet these high volumes, modern computing best practice generally requires adding more servers."

"SSL stands for Secure Sockets Layer and, in short, it's the standard technology for keeping an internet connection secure and safeguarding any sensitive data that is being sent between two systems, preventing criminals from reading and modifying any information transferred, including potential personal details."

Transport Layer Security (TLS) encrypts data sent over the Internet to ensure that eavesdroppers and hackers are unable to see what you transmit which is particularly useful for private and sensitive information such as passwords, credit card numbers, and personal correspondence. 

**Functions of Load balancer**
- Distributes client requests or network load efficiently across multiple servers
- Ensures high availability and reliability by sending requests only to servers that are online
- Provides the flexibility to add or subtract servers as demand dictates


## Project Requirements
- Amazon EC2 Instances (RedHat OS)
- Amazon EC2 Instance running Ubuntu OS to serve as our load balancer
- An existing three-tier achitecture to serve as web server and database server


![Project Architecture Diagram](https://darey.io/wp-content/uploads/2021/07/nginx_lb.png)

### Load Balancer Setup
- Launch an Ubuntu Server 20.04
- Allow traffic on port 80 for HTTP and 22 for SSH connection 

**Installing and configuring Nginx**

```bash
    sudo apt update -y
    sudo apt install nginx

    ##Configure Nginx
    sudo vi /etc/nginx/nginx.conf

    ##PAste the following into the **http** block of the configuration
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

    ##Confirm that the setting is OK/valid and restart nginx server
    sudo nginx -t
    sudo systemctl restart nginx
    sudo systemctl status nginx
```

**Configure SSL on the site**
- Register a new domain name 
- Associate an Elastic IP address with the Nginx server to persist the IP
- Update the A record on the DNS server of the domain name with the Elastic IP address
- Update nginx configuration with your domain name


**SSL certificate from Lets Encrypt**
- Install Certbot and it's dependencies for Nginx
- Set up Certbot and follow the prompt to create a free SSL certificate for your server
- Set up periodical renewal of your SSL/TLS certificate with a cronjob

```bash
    sudo apt install -y Certbot
    sudo apt install -y python-certbot-nginx

    #Setup Certbot on the nginx server
    sudo certbot --nginx

    ##Certbot dry run
    sudo certbot renew --dry-run

    sudo crontab -e

    ##Enter the following to make certbot certificate run every two days to renew the certificate
    * */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```
### Certbot
![Cerbot](https://github.com/scholarship-task/tooling/blob/master/project-10/screenshots/project10-certbot-status.png)
![Cerbot](https://github.com/scholarship-task/tooling/blob/master/project-10/screenshots/project10-certbot.png)
![Cerbot](https://github.com/scholarship-task/tooling/blob/master/project-10/screenshots/project10-certbot2.png)
![Cerbot](https://github.com/scholarship-task/tooling/blob/master/project-10/screenshots/project10-certbot-dry-run.png)
![Cronjob](https://github.com/scholarship-task/tooling/blob/master/project-10/screenshots/project10-cronjob.png)


## Screenshots
![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project-10/screenshots/project10-app-ui.png)

