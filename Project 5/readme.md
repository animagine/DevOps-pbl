# Implementing a client server architecture using mysql database management system 

## Step 1 : Setup servers

Provision 2 EC2 instances on AWS console. each must be Linux based.

Name the servers as 'mysql-server' and 'mysql-client' respectively

![Screenshot 2023-01-22 at 12 04 09 AM](https://user-images.githubusercontent.com/1076924/213890849-a22f1c9d-d806-4e5d-a3e3-81cd43bd39cb.png)

On the 'mysql server' install **MySQL server**.

`sudo apt update`

`sudo apt install mysql-server`

Open port 3306 on the inbound rules for the security group. Allow access only to the IP address of the 'mysql client' server

![Screenshot 2023-01-21 at 8 31 48 PM](https://user-images.githubusercontent.com/1076924/213890314-99ba0c4e-478f-43d6-ab39-5096b4211b95.png)

---

On the 'mysql server' edit the configuration file.

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

Replace '127.0.0.1' to '0.0.0.0.0' . This will allow the MySQL server to allow connections from remote hosts.

### install MySQL client on the 'mysql client' server

<img width="963" alt="Screenshot 2023-01-21 at 8 35 50 PM" src="https://user-images.githubusercontent.com/1076924/214049988-da764c28-3efc-4802-af66-a07e948bdac8.png">

From the "mysql client" server, connect to the "mysql server" database engine remotely without using ssh. Connection must be done using the mysql utility to perform the action.

<img width="1052" alt="Screenshot 2023-01-21 at 11 49 20 PM" src="https://user-images.githubusercontent.com/1076924/214051277-89b397aa-631d-419b-96a6-e70acb9d8e43.png">

