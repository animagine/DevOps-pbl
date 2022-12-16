#STEP 1 -After setting up new EC2 instance, run updates and install nginx and verify it installs successfully



sudo apt update
sudo apt install nginx

#check status of nginx server
sudo systemctl status nginx


#check that nginx is accessible locally 
sudo systemctl status nginx

curl http://localhost:80

#check that the serve can process requests via the internet
<ip address>>:80


## STEP 2 - installing MySQL ##

sudo apt install mysql-serverl mysql-server
sudo mysql

#secure root user password
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';


## STEP 3 - installing php ##
sudo apt install php-fpm php-mysql


# Configure nginx to use php 

sudo mkdir /var/www/projectLEMP
sudo chown -R $USER:$USER /var/www/projectLEMP
sudo nano /etc/nginx/sites-available/projectLEMP

------------------

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

----------------------------------

## STEP 4 activate nginx configuration located in the nginx sites-enabled directory ##
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

sudo systemctl reload nginx

#disable the default nginx host file
sudo unlink /etc/nginx/sites-enabled/default

#testing if the server works by creating an html file in the root directory

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 
'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

## STEP 5 - testing php with nginx to see if nginx can properly hand off .php files to the php processor##

sudo nano /var/www/projectLEMP/info.php

------------

<?php
phpinfo();


------------

# remove file after checking information
sudo rm /var/www/your_domain/info.php


## STEP 6 - retrieving data from sql database using php ##

sudo mysql -p
mysql> CREATE DATABASE `example_database`;
mysql> CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
mysql>exit

#testing user permission
mysql -u example_user -p

#once in the mysql console query the database structure
mysql> show databases;

#creating a todo_list
CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );

#test the table by inserting a feew rows of content and then test to see that data entry was successful
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
mysql>  SELECT * FROM example_database.todo_list;

#after confirmation is successful create a php script that will connect to the MySQL database
nano /var/www/projectLEMP/todo_list.php


---------------

<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}


---------------

#view php script in browser
http://<IP_or_public_domain>/todo_lidt.php
