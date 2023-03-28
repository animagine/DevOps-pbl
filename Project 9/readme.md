# Tooling website deployment automation with continous integration using JENKINS

![add_jenkins](https://user-images.githubusercontent.com/1076924/228161801-50f37fda-e0d0-4af9-ae4c-13dc67d393ab.png)


- First we need to Launch an EC2 instance running ubuntu server 20 and name it Jenkins. 
After doing this we go ahead to install the JDK . 
Jenkins is a Java-based application, that is why we need to install this dependency.

`sudo apt update`

`sudo apt install default-jdk-headless`

- Now we install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/etc/apt/sources.list.d/jenkins.list'
    
```

`sudo apt update`

`sudo apt-get install jenkins`

- Ensure that Jenkins is running.

`sudo systemctl status jenkins`

![Screenshot 2023-03-28 at 8 50 04 AM](https://user-images.githubusercontent.com/1076924/228166358-a8e04a8f-638e-4e9a-9974-55c1121662dc.png)

- Setup a new inbund rule on your security group which allows traffic through TCP port 8080 on your EC2 server.


- Perform initial setup fro jenkins from your browser

`http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`

![Screenshot 2023-03-28 at 8 54 57 AM](https://user-images.githubusercontent.com/1076924/228167743-58c3d397-eb03-483b-bdb5-b4a573a027d8.png)


In order to retrieve the administrator password, run this command in the terminal

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Once pasword has been entered, install the suggested plugins.

- Follow the setup wizard and create first Admin user

## Configuring jenkins to retrieve source codes from Github using Webhooks

- We need to enable Webhooks on Github repository





