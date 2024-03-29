# Load Balancer solution with Nginx and SSL/TLS

For this project I want to achieve two things.

1.  Configure Nginx as a load balancer
2.  REgister a new domain name and configure a secured connection using SSL/TLS certificates

![nginx_lb](https://user-images.githubusercontent.com/1076924/228246734-47da8deb-5d4b-4c35-85f3-22f0dc43d16a.png)


Because we previously had Apache2 running on our load balancer server, it has to be removed and replaced with nginx

`sudo apt update && sudo apt install nginx -y `
`sudo systemctl enable nginx'
`sudo systemctl start nginx'

Then we need to configure Nginx LB using the webserver's name defined in /etc/hosts

`sudo vi /etc/nginx/nginx.conf`

```
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;

![Screenshot 2023-03-29 at 2 30 46 PM](https://user-images.githubusercontent.com/1076924/228620954-e2a0e58a-7a39-4d3b-bf9a-11629162043e.png)


```
After doing this we need to restart the nginx service and also ensure that it is running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```


## Register a new domain name
We need to register a new domain name and configure a secured connection using SSL/TLS certificates.

Once the domain name has been acquired, on the aws console, search for "Route 53" and then create a new hosted zone.

-  Include the domain name and also set route traffic type to "public hosted zone". then click create
-  Change the nameservers on the domain service provider to that of the hosted zone. 
-  Create 2 "A - records", inlcude the public IP address of the Nginx Load Balancer

![Screenshot 2023-03-29 at 1 20 35 PM](https://user-images.githubusercontent.com/1076924/228619081-858fc761-b33f-4cc6-a9f4-7e208495174e.png)

![Screenshot 2023-03-29 at 1 23 12 PM](https://user-images.githubusercontent.com/1076924/228620497-c5d55c6d-9852-4f38-8b2b-a4de8ee3d93f.png)

![Screenshot 2023-03-29 at 1 43 17 PM](https://user-images.githubusercontent.com/1076924/228620539-5016f05c-2973-4fd7-8d06-c653f1526e15.png)

![Screenshot 2023-03-29 at 1 43 56 PM](https://user-images.githubusercontent.com/1076924/228620580-6c0a40e2-8df3-4db5-b9f2-cb296981f06a.png)

## continue configuration of reverse proxy on the Nginx config file

- We need to remove the default page in the nginx directory

`sudo rm -f /etc/nginx/sites-enabled/default`

- We need to check if nginx is successfully configured using the following command

 `sudo nginx -t`

 - Change directory to the nginx/sites-enabled directory

`cd /etc/nginx/sites-enabled/`

- Then run the following command within that directory

`sudo ln -s ../sites-available/nginx.conf`

- Reload nginx service.

` sudo systemctl reload nginx`

-  Enter domain name in browser to test configuration
-  
![Screenshot 2023-03-29 at 9 32 03 PM](https://user-images.githubusercontent.com/1076924/228660528-ec29ffb2-b030-4f8b-ad0b-2c322f16c11a.png)

## Installing Certbot certificate for ssl

` sudo apt install certbot -y`

We also need to install a module that the certbot depends on to run effectively

`sudo apt install python3-certbot-nginx -y`

check nginx syntax and then reload the nginx service.

`sudo nginx -t && sudo nginx -s reload`

## create ssl certificate for domain name

`sudo certbot --nginx -d mruforious.com.ng -d www.mruforious.com.ng`

![Screenshot 2023-03-29 at 10 05 51 PM](https://user-images.githubusercontent.com/1076924/228667504-b44aa020-cf28-4cf5-937d-dd458191ea49.png)

-  Reload the browser and see the changes 

![Screenshot 2023-03-29 at 10 33 33 PM](https://user-images.githubusercontent.com/1076924/228672903-a079d010-5ee7-486f-aeb2-7fcdb352657d.png)


## setup procedural renewal of ssl certificate

`sudo certbot renew --dry-run`

Add the following line

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`


