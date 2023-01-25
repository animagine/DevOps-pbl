# STEP1 - installing Apache2 and updating the firewall

- update a list of packages in package manager
`sudo apt update`

<img width="833" alt="Screenshot 2023-01-25 at 3 42 06 PM" src="https://user-images.githubusercontent.com/1076924/214605786-9bea12a0-58f4-4a0d-9064-01edabb49922.png">

<img width="830" alt="Screenshot 2023-01-25 at 3 42 41 PM" src="https://user-images.githubusercontent.com/1076924/214605968-707373a5-f606-477d-9c9e-252d19d25709.png">



- run apache2 package installation

`sudo apt install apache2`

<img width="832" alt="Screenshot 2023-01-25 at 3 43 59 PM" src="https://user-images.githubusercontent.com/1076924/214607482-6118b79f-c7f8-4dba-8f05-e798fae54b7c.png">

- check apache2 status

`sudo systemctl status apache2`

<img width="828" alt="Screenshot 2023-01-25 at 3 45 43 PM" src="https://user-images.githubusercontent.com/1076924/214607636-7dfa5199-a3cf-4822-b37c-a0ab052a4f7f.png">


- check if server can be accessed locally on machine

`curl http://localhost:80'

<img width="921" alt="Screenshot 2023-01-25 at 3 47 05 PM" src="https://user-images.githubusercontent.com/1076924/214607972-15281a64-52ae-44c1-a72c-2ca1b05148f5.png">


- test apache2's response to request on the internet via browser

`http://<insert public IP address from aws dashboard>:80`

<img width="1464" alt="Screenshot 2023-01-25 at 3 48 50 PM" src="https://user-images.githubusercontent.com/1076924/214608724-5131d763-5c90-44b6-92cd-2b256a832ea3.png">



# STEP 2 - installing MySQL 

- installing MySQL
```
sudo apt install mysql-server 
sudo mysql

```
<img width="924" alt="Screenshot 2023-01-25 at 4 01 35 PM" src="https://user-images.githubusercontent.com/1076924/214609784-22c91f74-834a-4666-bac4-41f661e47c55.png">


- secure root user password

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`


# STEP 3 installing PHP 

```
sudo apt install php libapache2-mod-php php-mysql
php -v

```
<img width="920" alt="Screenshot 2023-01-25 at 4 04 36 PM" src="https://user-images.githubusercontent.com/1076924/214610213-15430344-6fa6-4c72-8f9c-213938dd7c7b.png">

# STEP 4 creating a virtual host on apache server


- creating a new directory and changing file ownership

`sudo mkdir /var/www/projectlamp`

`sudo chown -R $USER:$USER /var/www/projectlamp`

`sudo vi /etc/apache2/sites-available/projectlamp.conf`

---

#inputing basic configuration for the server

```
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

# Enabling the new virtual host and disabling the default apache2 website

`sudo a2ensite projectlamp`

`sudo a2dissite 000-default`

- Testing to see if the configuration works without errors

`sudo apache2ctl configtest`

`sudo systemctl reload apache2`

# creating a new file index.html in the directory

```
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname)
'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html

```

# STEP 5 - Enabling PHP on the new site

- changing the default root file from index..html to index.php

`sudo vim /etc/apache2/mods-enabled/dir.conf`

- replacing the content of the config file to reflect the changes

```
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>

```
---
-  reloading apache2 to reflect the changes

`sudo systemctl reload apache2`




- adding index..php to the root folder

`vim /var/www/projectlamp/index.php`



```

<?php
phpinfo();

```
<img width="1464" alt="Screenshot 2023-01-25 at 4 21 29 PM" src="https://user-images.githubusercontent.com/1076924/214608952-62632454-4cc8-4d75-87be-c27ef0245dca.png">


#for security reasons, after checking the information of the php version installedd on your server, it should be removed

`sudo rm /var/www/projectlamp/index.php`
