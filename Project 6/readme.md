# Web solution with wordpress

1.  I will be creating two EC2 instances named "Web server" and "DB server" respectively. Both instances will have "Redhat linux " running on them.
for the "Web server" instance, I will be creating 3 additional volumes and attacheing them manually to my instance.

In my Linux terminal, I will begin configuration by doing the following after connecting to my instance using ssh.

`lsblk`

![Screenshot 2023-01-26 at 2 31 30 PM](https://user-images.githubusercontent.com/1076924/214848003-3350557d-dd20-444d-ad40-df4ee2e089a0.png)


2.  After inspecting and viewing the block devices attached to my "Web server" use the "gdisk" utility to create a single partition on each of the 3 disk volumes.

`sudo gdisk /dev/xvdf`


![Screenshot 2023-01-26 at 2 38 08 PM](https://user-images.githubusercontent.com/1076924/214849573-0d7d3077-2813-49fd-8a37-ec2af06679b1.png)

3.  Using the "lsblk" utilty, I will inspect the newly configured partition of the 3 disk volumes.

`lsblk`

![Screenshot 2023-01-26 at 2 44 12 PM](https://user-images.githubusercontent.com/1076924/214850747-80cdb240-e56d-4c1e-924e-222c1f632bf5.png)

4.  I will proceed to install "lvm2" package and "lvmdiskscan" to check available partitions.

`sudo yum install lvm2`

'sudo lvmdiskscan`

![Screenshot 2023-01-26 at 2 47 45 PM](https://user-images.githubusercontent.com/1076924/214851963-dea669ed-6d29-4993-818b-c994cbe3c6f5.png)

5.  I will mark the 3 disks as physical volumes to be used by the "lvm" using the following command

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

```
6.  Verify that the physical volumes have been created successfully

`sudo pvs`

![Screenshot 2023-01-26 at 2 56 26 PM](https://user-images.githubusercontent.com/1076924/214853904-043f966e-6c93-499d-a7d9-5860e896e156.png)

7.  All 3 volumes need to be added to a volume group (VG). Uing the "vgcreate" utility and nameing the group "webdata-vg"

`sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

8.  I will verify that the volume group was created successfully using `sudo vgs` and then create 2 logical volumes using the "lvcreate" utility. The volumes will be named "apps-lv" and "logs-lv" respectively.

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg

```
9.  Verify that the logical volume was created succesfully by running

`sudo lvs`

![Screenshot 2023-01-26 at 3 40 53 PM](https://user-images.githubusercontent.com/1076924/214864614-c0b9a047-5113-47ef-946a-781c429efc73.png)

10. Verify the entire setup using the following command

`sudo vgdisplay -v #view complete setup - VG, PV, and LV'

<img width="853" alt="Screenshot 2023-01-26 at 3 45 53 PM" src="https://user-images.githubusercontent.com/1076924/214866274-124e2dc6-f378-476a-a669-dad073d2e731.png">


`sudo lsblk`

<img width="850" alt="Screenshot 2023-01-26 at 3 48 14 PM" src="https://user-images.githubusercontent.com/1076924/214866494-fb537ae1-93fc-4714-872e-e124d76af9ec.png">

11. The logical volumes need to be formatted to reflect a files system. For the purpose of this project I will be suing the ext4 files system. I will format the files the volumes using the following commands in the termianal.

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

```
12. I will need a directory to directory to store website files and also backup log data. To do that, i will be creating those directories in the termianal

```

sudo mkdir -p /var/www/html

sudo mkdir -p /home/recovery/logs

```
13. Here i will mount /var/www/html unto the apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

14. Using the "rsync" utility I will  backup all the files in the log directory /var/log into /home/recovery/logs

`sudo rsync -av /var/log/. /home/recovery/logs/`

<img width="851" alt="Screenshot 2023-01-26 at 4 52 51 PM" src="https://user-images.githubusercontent.com/1076924/214883261-fe44d8ea-46d2-4a62-a58e-f1ad41613d6e.png">

15. Will also be mounting /var/log on thee logs-lv logical volume

`sudo mount /dev/webdata-vg/logs-lv /var/log`

16. Restore the log files back into the /var/log directory

`sudo rsync -av /home/recovery/logs/. /var/log`

-------

## Updating the 'ETC/FSTAB' FILE

Good progress has been made! But there's more to be done.

The UUID of the devices will be used to update the etc/fstab file in the folowing manner

`sudo blkid`

<img width="858" alt="Screenshot 2023-01-26 at 5 13 17 PM" src="https://user-images.githubusercontent.com/1076924/214891611-eafccb9a-974a-46e8-883d-6eac4a890294.png">

`sudo vi /etc/fstab`

17. Test configuration and reload daemon

`sudo mount -a`

`sudo systemctl daaemon-reload`

<img width="555" alt="Screenshot 2023-01-26 at 9 26 25 PM" src="https://user-images.githubusercontent.com/1076924/214943249-57a5c460-1782-446d-ace0-0d9cde74a00c.png">

18. Having done that, I will verify the setup by runnung 

`dh -f`

## Preparing DB server

19..  I will be setting up this server with the same configuration as the web server. Saave for a few differences. A DB volume that will contain all the data for the database. I will mount this volume on the /db directory.


<img width="721" alt="Screenshot 2023-01-26 at 11 25 17 PM" src="https://user-images.githubusercontent.com/1076924/215357676-98f0c3d9-0f3f-46b7-8c8a-740278a23300.png">

## Installing WordPress

20. Update the server's filesystem, this in turn updates the repository

`sudo yum -y update`

21. Install wget, Apache and it's dependencies. 

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

22. Staart Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd

```

23. Install Php and it's dependencies

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1

```

<img width="1026" alt="Screenshot 2023-01-29 at 11 08 52 PM" src="https://user-images.githubusercontent.com/1076924/215358379-a630c34f-22fc-4dbc-96bc-be0b2964084a.png">


24. Restart Apache

`sudo systemctl restart httpd'

25. Download WordPress and copy it to the /var/www/html directory

```
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
  
 ```
26. Configure SELinux policies

```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1

```
<img width="787" alt="Screenshot 2023-01-29 at 11 30 39 PM" src="https://user-images.githubusercontent.com/1076924/215359340-1f1f247e-f71c-4ec8-96dc-500ddfa51290.png">


## Install mysql on DB server

```
sudo yum update
sudo yum install mysql-server
```

Restart server after installation is complete

```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
<img width="723" alt="Screenshot 2023-01-29 at 11 41 06 PM" src="https://user-images.githubusercontent.com/1076924/215359924-a43152b6-2dfc-4960-8a6e-615b32e4adab.png">

## Configure DB to work with WordPress

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL PRIVILEGES ON *.* TO `myuser`@`<web-server-private-IP-Address>`WITH GRANT OPTION;
FLUSH PRIVILEGES;
SHOW DATABASES;
exit

```
## 
