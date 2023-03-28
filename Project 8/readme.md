# Load balancer solution with Apache


![project8_final](https://user-images.githubusercontent.com/1076924/228159973-427c2473-88f9-4da7-9fc1-ff4b607c38e2.png)

We will need to create an Ubuntu Server EC2 instance which will serve as a load balancer and open up TCP port 80.

We will also need to install Apache and configure it to pint all traffic coming to the Load Balancer to both webservers.

```
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

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

![Screenshot 2023-03-28 at 12 01 36 AM](https://user-images.githubusercontent.com/1076924/228086472-d1d05cd0-14e0-4d14-bf43-0d65b5750718.png)


After entering these instructions, ensure that Apache2 is running. 

`sudo systemctl status apache2`

![Screenshot 2023-03-28 at 12 02 35 AM](https://user-images.githubusercontent.com/1076924/228086316-9ed0420e-fbb1-43cc-bfdc-3fd6413b3bd6.png)

Next, we configure Load balancing.

`sudo vi /etc/apache2/sites-available/000-default.conf`

We add the following instructions in the <VirtualHost *:80> </VirtualHost>


```
<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

```

![Screenshot 2023-03-28 at 12 13 14 AM](https://user-images.githubusercontent.com/1076924/228087507-022a5022-0c7d-419a-8ef4-e5a71a39052e.png)

After making these changes, we can verify that it works by trying to accessthe Load Balancer's public IP address or Public DNS names from the browser.

`http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`

<img width="977" alt="Screenshot 2023-03-28 at 8 02 07 AM" src="https://user-images.githubusercontent.com/1076924/228157907-e6ca0b3c-1efa-4a17-9613-a376953a253b.png">

## Configuring Local DNS names resolution

We need to open up the hosts file

`sudo vi /etc/hosts`

And then add 2 records into this file with local IP addresses of arbitrary names for both webservers

```
<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2

```
![Screenshot 2023-03-28 at 8 22 18 AM](https://user-images.githubusercontent.com/1076924/228159499-34641a5f-4831-4fc9-bd54-61301ff8006b.png)


