# Devops tooling website solution
In this project i will be implementing a solution that consists of the following features

1. An infrastructure setup on AWS
2. A Webserver with Redhat Enterprise Linux 8
3. A database server with Ubuntu and Mysql
4. A storage server : Red hat enterprie 9 + NFS server
5. Programming language: PHP
6. Code repository: github

---

## Step 1 : Preparing NFS servers

First things first, i will be creating 4 webservers running RHEL (Redhat Enterprise linux) on AWS and then another server for the Database running an Ubuntu OS.

<img width="835" alt="Screenshot 2023-02-06 at 9 36 32 PM" src="https://user-images.githubusercontent.com/1076924/217079631-e5d215b0-edd4-41b5-8377-846f1b9d988e.png">

I will name one of the RHEL instances "NFS", then proceed to creating and attaching 3 EBS volumes to it.
They volumes will be named "lv-opt" , "lv-apps", and "lv-logs" respectively. The disk volumes will also be formatted to "xfs".

<img width="873" alt="Screenshot 2023-02-06 at 9 50 10 PM" src="https://user-images.githubusercontent.com/1076924/217082641-8b8cc22b-0d4c-4c8e-a694-2f5db84d0a18.png">

I will now access the NFS server via my terminal. Then list all the disks available on the server.

`lsblk`

<img width="491" alt="Screenshot 2023-02-06 at 9 57 06 PM" src="https://user-images.githubusercontent.com/1076924/217083903-9d28646d-c701-48ae-92c3-b7788dd43456.png">

Using the "gdisk" utility, i will create 3 partitions.

`sudo gdisk /dev/xvdf`

`sudo gdisk /dev/xvdg`

`suudo gdisk /dev/xvdh`

<img width="613" alt="Screenshot 2023-02-06 at 10 01 30 PM" src="https://user-images.githubusercontent.com/1076924/217085184-5aa7f873-bff4-495c-a873-0835511326d2.png">

I will confirm my configuration by using `lsblk` command again.

<img width="505" alt="Screenshot 2023-02-06 at 10 03 43 PM" src="https://user-images.githubusercontent.com/1076924/217086432-90470c8d-69f4-403a-9685-a59e8e779104.png">

Having configured and created the partitions successfully, I will install the "Lvm package"

`sudo yum install lvm2 -y`

<img width="681" alt="Screenshot 2023-02-06 at 10 09 56 PM" src="https://user-images.githubusercontent.com/1076924/217088895-0dafabe2-00f4-4784-978e-40cd303c0628.png">

Will check for available partitions using

`sudo lvmdiskscan`

<img width="389" alt="Screenshot 2023-02-06 at 10 13 46 PM" src="https://user-images.githubusercontent.com/1076924/217089484-9f4b9331-8919-4f73-aa3b-90837b72d00e.png">

and also create physical volumes for each of the disks that has been created.

`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

Confirm that the volumes were created successfully using `sudo pvs`

<img width="433" alt="Screenshot 2023-02-06 at 10 19 02 PM" src="https://user-images.githubusercontent.com/1076924/217090466-a8309968-a2bb-4a4b-9878-522eeff8ae8b.png">

Now i will create a volume group to house the 3 physical volumes.

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

`sudo vgs`

<img width="660" alt="Screenshot 2023-02-06 at 10 24 01 PM" src="https://user-images.githubusercontent.com/1076924/217091706-f2f1b05d-b71c-4e1f-bee3-c7c8679776ec.png">

- Creating logical volumes

`sudo lvcreate -n lv-apps -L 9G webdata-vg`
`sudo lvcreate -n lv-logs -L 9G webdata-vg`
`sudo lvcreate -n lv-opt -L 9G webdata-vg`

<img width="662" alt="Screenshot 2023-02-06 at 10 33 52 PM" src="https://user-images.githubusercontent.com/1076924/217092941-0830b566-c6d2-41bd-b125-9b431a7235a6.png">

I will format the disks as "xfs"

`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`
`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`
`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`


<img width="573" alt="Screenshot 2023-02-06 at 10 40 40 PM" src="https://user-images.githubusercontent.com/1076924/217094193-3171931a-0b6f-4999-8f0e-b37025b17ed1.png">

- create mount points for "lv-apps","lv-logs" and "lv-opt"

`sudo mkdir /mnt/apps`
`sudo mkdir /mnt/logs'
`sudo mkdir /mnt/opt`

- Mount them using

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`
`sudo mount /dev/webdata-vg/lv-apps /mnt/apps'


<img width="685" alt="Screenshot 2023-02-06 at 10 46 27 PM" src="https://user-images.githubusercontent.com/1076924/217095167-613390d0-1d35-4a8a-bb8a-ba6fab6b47b0.png">

- install NFS server
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service

```

<img width="684" alt="Screenshot 2023-02-06 at 10 58 33 PM" src="https://user-images.githubusercontent.com/1076924/217097204-d2d6ec1d-f213-41c2-bbc6-0d2e1d5e1bc2.png">


Now, I will setup permissions that will allow the webservers to read,write and execute files on the NFS

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

```
- Restart the nfs-server service

`sudo systemctl restart nfs-server.service`

- Configure access to NFS for clients within the same subnet

```
sudo vi /etc/exports

#paste the code below in the file, save and close

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)


```
`sudo exportfs -arv`

<img width="595" alt="Screenshot 2023-02-07 at 12 42 11 AM" src="https://user-images.githubusercontent.com/1076924/217112005-727aa541-d7a4-4f3f-983e-3cfe3918df7a.png">

Will need to check which ports are used by the NFS open it using security groups. The ports to be opened are TCP 111, UDP 111, UDP 2049

`rpcinfo -p | grep nfs`

---

## Configuring Database on the databse server (Ubuntu)
 
 -  I will install mysql on the database server, create a database and name it  "tooling".
 -  Will create a database user and name it "webaccess"
 -  Grant permission to the "webaccess" user on the "tooling" database to be able to do anything only from  
    the webservers "subnet cidr"
 
 `sudo apt -y update`
 
 `sudo apt install mysql-server -y
 
 `sudo mysql`
 
 `mysql>create database tooling;`
 
 `mysql>create user 'webaccess'@'<subnet-cidr>' identified by 'password';`
 
 `mysql>grant all privileges on tooling.* to 'webaccess'@'<subnet-cidr>';`
 
 `mysql> flush privileges;`
 
 `mysql> show databases;`
 
 <img width="805" alt="Screenshot 2023-02-07 at 11 12 54 PM" src="https://user-images.githubusercontent.com/1076924/217378697-821896a1-fada-43e4-9961-5d41fbcee8ab.png">

----

## Preparing the webservers

- Connect to the Webserver 1
`sudo yum update -y`

- Install NFS client

`sudo yum install nfs-utils nfs4-acl tools -y`

- Mount /var/www and target the NFS server's export for apps


```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

```
Verify the mount status of the NFS server by running `df -h`

<img width="531" alt="Screenshot 2023-02-08 at 12 27 30 AM" src="https://user-images.githubusercontent.com/1076924/217390204-c0bc5ddb-ce07-480f-b344-4075843b1827.png">

Having done that, open /etc/fstab and ensure the changes persist after reboot

`sudo vi /etc/fstab`

`<NFS-server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`




- Install Remi's repository, Apache and php

```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1

```

**This procedures will be repeated on the other 2 Webservers**

-----

- Locate the log folder for Apache on the Web server and mount it to the NFS server's export for logs.

`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd`

<img width="491" alt="Screenshot 2023-02-08 at 1 52 00 AM" src="https://user-images.githubusercontent.com/1076924/217400812-eaa13c5a-441a-4755-b0ad-288ea018b5bf.png">



`sudo vi /etc/fstab`

`<NFS-server-Private-IP-Address>:/mnt/logss /var/log/httpd nfs defaults 0 0`

- check that the mount was a succeess by using th 'df -h' command.

<img width="491" alt="Screenshot 2023-02-08 at 1 52 00 AM" src="https://user-images.githubusercontent.com/1076924/217401401-e0387ac6-9862-418c-a78a-22e34ef68e1d.png">

---

- Fork this "tooling" repo from [Darey.io Github Account](https://github.com/darey-io/tooling.git)

- Additionally copy the contents of the html folder in the "tooling" directory to the /var/www/html directory

`cd tooling`
`sudo cp -R html/. /var/wwww/html`

<img width="984" alt="Screenshot 2023-02-08 at 2 13 53 AM" src="https://user-images.githubusercontent.com/1076924/217403305-044e5d7d-70a1-490b-aec5-6e736d000252.png">

- Once that is done update the website's configuration to connect to the database by editiing the functions.php file in /var/www/html/functions.php.
Apply tooling-db.sql script to the database using this command 

`mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

- Open the website in your browser 

`http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php`

<img width="666" alt="Screenshot 2023-02-08 at 10 10 37 PM" src="https://user-images.githubusercontent.com/1076924/217832461-713277ba-5d38-4384-81ba-7b245510300b.png">

-  login with username and password.

![Screenshot 2023-02-09 at 2 38 20 PM](https://user-images.githubusercontent.com/1076924/217832572-600e5ce9-183e-48c0-8dea-cb1716c3b3ba.png)




