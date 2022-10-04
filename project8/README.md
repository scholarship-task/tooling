# Load Balancer Solution with Apache

This project add a load-balancer to the existing architecture so that the user will only be able to access the service using only one IP address. 
"Load balancing refers to efficiently distributing incoming network traffic across a group of backend servers, also known as a server farm or server pool.

Modern high‑traffic websites must serve hundreds of thousands, if not millions, of concurrent requests from users or clients and return the correct text, images, video, or application data, all in a fast and reliable manner. To cost‑effectively scale to meet these high volumes, modern computing best practice generally requires adding more servers."

**Functions of Load balancer**
- Distributes client requests or network load efficiently across multiple servers
- Ensures high availability and reliability by sending requests only to servers that are online
- Provides the flexibility to add or subtract servers as demand dictates


## Project Requirements
- Amazon EC2 Instances (RedHat OS)
- Amazon EC2 Instance running Ubuntu OS to serve as our load balancer
- An existing three-tier achitecture to serve as web server and database server


![Project Architecture Diagram](https://darey.io/wp-content/uploads/2021/07/Tooling-Website-Infrastructure-wLB.png)

### EC2 Instance List 

![Project Architecture Diagram](https://github.com/scholarship-task/tooling/blob/master/project8/screenshots/project8-ec2-list.png)

### Load Balancer Setup
- Launch an Ubuntu Server 20.04
- Allow traffic on port 80 for HTTP and 22 for SSH connection 

**Installing Apache and Apache Lad Balancer module**

```bash
    sudo apt update -y
    sudo apt install apache2 -y
    sudo apt-get install libxml2-dev -y

    #Enable following modules:
    sudo a2enmod rewrite
    sudo a2enmod proxy
    sudo a2enmod proxy_balancer
    sudo a2enmod proxy_http
    sudo a2enmod headers
    sudo a2enmod lbmethod_bytraffic

    #Restart apache2 service
    sudo systemctl restart apache2
```

**Configure Load Balancing**


Paste the following into the /etc/apache2/sites-available/000-default.conf
This must be in the <VirtualHost> </VirtualHost> block
```
    <Proxy "balancer://mycluster">
               BalancerMember http://172.31.28.30:80 loadfactor=5 timeout=1
               BalancerMember http://172.31.18.58:80 loadfactor=5 timeout=1
               BalancerMember http://172.31.20.169:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
    </Proxy>

    ProxyPreserveHost On
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
```
### Configuration file
![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project8/screenshots/project8-loadbalancer-config.png)

**Restart Apache Server to apply the settings**
```
    sudo systemctl restart apache2
```

Access the site using the public IP address of the Apache Server


### Configure Local DNS Names Resolution
Edit the /etc/hosts file with the IP address of the webserver

```bash
    sudo vi /etc/hosts

    <WebServer1-Private-IP-Address> Web1
    <WebServer2-Private-IP-Address> Web2
    <WebServer2-Private-IP-Address> Web2
```

Apache conf file can be modified as following and using curl to do local testing on the terminal

```

    <Proxy "balancer://mycluster">
               BalancerMember http://Web1:80 loadfactor=5 timeout=1
               BalancerMember http://Web2:80 loadfactor=5 timeout=1
               BalancerMember http://Web3:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
    </Proxy>

    ProxyPreserveHost On
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/

    ##Curl command
    curl web1
```

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project8/screenshots/project8-named-host.png)

## Screenshots
![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project8/screenshots/project8-ui-dashboard.png)

![App Screenshots](https://github.com/scholarship-task/tooling/blob/master/project8/screenshots/project8-app-ui.png.png)

## Final Architecture Diagram with Apache Load Balancer
![Apache Load Balancer](https://darey.io/wp-content/uploads/2021/07/project8_final.png)


