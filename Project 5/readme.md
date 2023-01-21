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
