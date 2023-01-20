# MEAN stack implementation on Ubuntu in AWS

**STEP 1: Install Nodejs** 

Spin up an ubuntu server EC2 instance in aws and update it.

`sudo apt update`

`sudo apt upgrade`

<img width="886" alt="Screenshot 2023-01-20 at 2 09 10 PM" src="https://user-images.githubusercontent.com/1076924/213704118-c8c28f02-ef0b-45eb-9b86-1fdceb9fb1f6.png">


Once this is completed, proceed to adding certificates and then install nodejs

`sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`

`curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`

<img width="890" alt="Screenshot 2023-01-20 at 2 17 56 PM" src="https://user-images.githubusercontent.com/1076924/213704019-08a80759-d8c4-4106-a0ce-9656df0b76ae.png">

**Install Nodejs**

`sudo apt install -y nodejs`

**STEP 2: Install MOngoDB**

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

`sudo apt install -y mongodb`

running the script above may flag an error:

> The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but it is not installable
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but it is not installable
 mongodb-org-shell : Depends: libssl1.1 (>= 1.1.1) but it is not installable
E: Unable to correct problems, you have held broken packages.


but should be easily fixed following the instructions below

**N.B - As at writing this, MongoDb has no official build for ubuntu 22.04 at the moment
Ubuntu 22.04 has upgraded libssl to 3 and does not propose libssl1.1
You can force the installation of libssl1.1 by adding the ubuntu 20.04 source:**

```
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt-get update
sudo apt-get install libssl1.1

```

Install mongodb again

`sudo apt install -y mongodb`

Delete the focal security that was created

`sudo rm /etc/apt/sources.list.d/focal-security.list`

once this is done, start the server

`sudo service mongod start`

verify that the service is up and running

`sudo systemctl status mongod`

<img width="882" alt="Screenshot 2023-01-20 at 3 19 22 PM" src="https://user-images.githubusercontent.com/1076924/213721154-8dfba814-f750-4cda-8b63-e9ec7fd727e1.png">

Install npm (Node Package Manager)
