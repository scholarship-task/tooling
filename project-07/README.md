
# DevOps Tooling Website Solution

This project implements a three-tier architecture extending the previous project by making use of Network File System to share files between client and server. There are 3 client connected to a Network File System and also a Database server running on Ubuntu 20.04.


## Project Requirements
- Amazon EC2 Instances (RedHat OS)
- Amazon EBS attached to the EC2 Instances
- Storage Server running on Red Hat OS (Rhel 8)
- MySQL Database server running on Ubuntu 20.04
- Security Group to allow connection on port 22 for SSH, 80 for HTTP connections, port 111, 2049 for UDP and TCP connection, and port 3306 on the database server for MySQL.


![Project Architecture Diagram](https://www.darey.io/wp-content/uploads/2021/07/Tooling-Website-Infrastructure.png)

### EC2 Instances

![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7--ec2-list.png)


-Create EC2 Instance for the server and attach 3 EBS volumes of 10GB capacity each to be used for app, log and opt data for the NFS Server.

### Configure the EC2 instances attachment as required for NFS server

The volumes need to be mounted and formatted to be able to access data from the instance. And they are always in the /dev/ folder. This folder contains all the file storage attached to the system.
This folder can be viewed by running **ls /dev/** or **lsblk** command.

The following steps will also be required on the two instances (WebServer and DBServer)

List all the attached volumes on the instances and noticed their names because the names will be needed when formatting the disk for usage

```bash
    lsblk
    ls /dev/
```

The **lsblk** command lists information about all available or the specified block
devices. The lsblk command reads the sysfs filesystem and udev db to gather information. If the udev db is not available or lsblk is
compiled without udev support, then it tries to read LABELs, UUIDs and
filesystem types from the block device. In this case root permissions
are necessary. Source: **man lsblk**

Use gdisk command to create a single partition on the volumes
gdisk is an interactive GUID partition table (GPT) manipulator and it requires root privilege. After entering the command enter **n** to create a new partition on the disk and **w** to save the changes. Repeat for the remaining two volumes. View the changes using **lsblk** command

```bash
    sudo gdisk /dev/xvdf
    sudo gdisk /dev/xvdg
    sudo gdisk /dev/xvdh
```

Install Logical Volume Manager to be able to create virtual block devices from block storage and run **lvmdiskscan ** to scan for disk that may be used as physical volumes
**YUM** and **DNF** are package managers for RedHat, CentOS and Fedora Linux distribution
```
sudo yum install lvm2
sudo lvmdiskscan
```

Use **pvcreate** utility to initialize a physical volume on a device so that it can be recognized as belonging to the Logical Volume Manager. A PV can be placed on a whole device or partition and use **pvs** command to display information about the physical volumes.


```
    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1

    sudo pvs
```

Create Logical Volume group with the following command **vgcreate** and verified that the logical volumes has been created with **vgs**

```
    sudo vgcreate nfsserver-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
    sudo vgs
```

Create Logical Volume from the Logical Volume group created above
The **lv-apps** is used for the application data while the **lv-logs** is used for the logging data and **lv-opt** to be used by Jenkins Server which will be implemented in a future project.
Verify that the logical volume is created with **lvs**
View the complete settings about the PV, LV, VG
```
    sudo lvcreate -n lv-apps -L 9G nfsserver-vg
    sudo lvcreate -n lv-logs -L 9G nfsserver-vg
    sudo lvcreate -n lv-opt -L 9G nfsserver-vg
    sudo lvs
    sudo vgdisplay -v #view complete setup - VG, PV, and LV
    sudo lsblk 
```
![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-lsblk.png)

Format the logical volumes with mkfs to build a linux file system, **xfs** is the file extension.

```
    sudo mkfs -t nfs /dev/nfsserver-vg/lv-apps
    sudo mkfs -t nfs /dev/nfsserver-vg/lv-logs
    sudo mkfs -t nfs /dev/nfsserver-vg/lv-opt
```

Create **/mnt/apps directory** to store app files

```
    sudo mkdir -p /mnt/apps
```

Create **/mnt/logs** to store backup of log data

```
    sudo mkdir -p /mnt/logs
```
Create **/mnt/opt** to store Jenkins data for a future project

```
    sudo mkdir -p /mnt/logs
```

Mount **directory** on the logical volume

```
    sudo mount /dev/nfsserver-vg/lv-apps /mtn/apps
    sudo mount /dev/nfsserver-vg/lv-logs /mtn/logs
    sudo mount /dev/nfsserver-vg/lv-opt /mtn/opt
```
![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-mount.png)

Update **/etc/fstab** file so that the mount configuration will persist after restart of the server.
The fstab file typically lists all available disks and disk partitions, and indicates how they are to be initialized or otherwise integrated into the overall system's file system.
Run **blkid** to print block block devices information, the most important information is the UUID. Each entry line in the fstab file contains six fields, each one of them describes a specific information about a filesystem.
- First field – The block device. ...
- Second field – The mountpoint. ...
- Third field – The filesystem type. ...
- Fourth field – Mount options. ...
- Fifth field – Should the filesystem be dumped ? ...
- Sixth field – Fsck order.

```
    sudo blkid
    sudo vim /etc/fstab
```

Sample /etc/fstab file

Test the configuration and reload the daemon

```
   sudo mount -a
   sudo systemctl daemon-reload
```

### Install NFS server

Network File Sharing (NFS) is a protocol that allows you to share directories and files with other Linux clients over a network. Shared directories are typically created on a file server, running the NFS server component. Users add files to them, which are then shared with other users who have access to the folder.An NFS file share is mounted on a client machine, making it available just like folders the user created locally. NFS is particularly useful when disk space is limited and you need to exchange public data between client computers.

```bash
    sudo yum -y update
    sudo yum install nfs-utils -y
    sudo systemctl start nfs-server.service
    sudo systemctl enable nfs-server.service
    sudo systemctl status nfs-server.service
```

Exports mounts for webservers and give the necessary permission and ownerships to the mount path
```bash
    sudo chown -R nobody: /mnt/apps
    sudo chown -R nobody: /mnt/logs
    sudo chown -R nobody: /mnt/opt

    sudo chmod -R 777 /mnt/apps
    sudo chmod -R 777 /mnt/logs
    sudo chmod -R 777 /mnt/opt

    sudo systemctl restart nfs-server.service
```
 ![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-nfs-server.png)
 ![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-nfs-server2.png)

**Configure access to NFS for clients within the same subnet**
Edit the /etc/exports file with the following:

```bash
    sudo vi /etc/exports

    ##Replace IP Address with the private IP address of the webservers
    /mnt/apps 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
    /mnt/logs 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
    /mnt/opt 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
    
    ##Format
    /mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
    /mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
    /mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

    sudo exportfs -arv
    rpcinfo -p | grep nfs
```

### Configure the Database server
Install MySQL server on Ubuntu 20.04 and ensure port 3306 is open to accept connection only from the Subnet

```bash
    sudo apt update -y
    sudo apt install -y mysql-server
    sudo systemctl restart mysql-service
    sudo systemctl enable mysql-service
    sudo systemctl status mysql-service

    ##Create database 
    sudo mysql
    CREATE DATABASE tooling;
    CREATE USER webaccess@<Web-Server-Private-IP-Address> IDENTIFIED BY 'password';
    GRANT ALL ON tooling.* TO webaccess@<Web-Server-Private-IP-Address>;
    FLUSH PRIVILEGES;
    SHOW DATABASES;
    exit
```

### Prepare the Web Servers (NFS Client)
Launch 3 EC2 instances running Red Hat OS (RHEL8)

Install NFS package on the 3-web Servers

```bash
    sudo yum install nfs-utils nfs4-acl-tools -y
```

Mount /var/www/ from the Web Server to the NFS Server and verify that it is successful

```
    sudo mkdir /var/www
    ##Sample
    sudo mount -t nfs -o rw,nosuid 172.31.29.152:/mnt/apps /var/www

    sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
    df -h
```
 ![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-mount-logs.png)
 ![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-mount-nfs.png)

Persist the changes using the **etc/fstab** file

```
    sudo vi /etc/fstab

    ##
    172.31.29.152:/mnt/apps /var/www nfs defaults 0 0

 ```
 ![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-etc-fstab.png)
 ![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-etc-fstab2.png)
 ### Install Apache and PHP on the servers   

**Install PHP and it's package dependencies**

```
    sudo yum install httpd -y
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
**Configure SELinux policy**

The SELinux Policy is the set of rules that guide the SELinux security engine. It defines types for file objects and domains for processes. It uses roles to limit the domains that can be entered, and has user identities to specify the roles that can be attained.

```
    sudo chown -R apache:apache /var/www/html
    sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R
    sudo setsebool -P httpd_can_network_connect=1
```

### Copy the project file

Use rsync command to copy the file from your local PC to any of the server and it will be available on all the servers

    rsync -a ~/tooling/  ec2-user@54.196.252.117:/var/www/
    
![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-tooling-rsync.png)

Update the **functions.php** file with the database credentials

![EC2 Instances](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-functions.php.png)


Pass the **tooling-db.sql** from the webserver granted access to communicate with the DB server

Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php to access the app

    mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql


## Screenshots

![App User Interface](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-app-ui.png)
![App User Interface](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-app-ui2.png)
![App User Interface](https://github.com/scholarship-task/tooling/blob/master/project-07/screenshots/project7-app-ui-3.png)


## Challenges

I had database connection error, I had to edit the functions.php with the IP address of the database server and also the username, password and the database.
I had trouble logging in to the website, I had to look through the code with a close eyes and hence find out that they password was encrypted with md5 algorithm and it has to be decrypted before I can login in to the system.

