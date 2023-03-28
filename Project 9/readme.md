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

-   We need to enable Webhooks on Github repository

-   Under the Github repository settings, go to the webhooks tab.

-   In the Payload field enter `http://<jenkins-server-public-ip-address>:8080/github-webhook/`

-   And for the event triggers of the webhook select the "Just the push event"
-   Also set the delivery of the hook when it is triggered to Active




![Screenshot 2023-03-28 at 9 11 50 AM](https://user-images.githubusercontent.com/1076924/228172320-5d786d99-8023-4842-a929-a14db71f227d.png)


-   On the Jenkins dashboard, select and create a new item. Name it and then set it as a freestyle project. We will need to provide the link to our GitHub repository and credentials so jenkins could access the files in the repository.

 **N.B  While setting this up, in order to avoid errors we need to configure jnkins global security to enable proxy compatbility.**

-   
![Screenshot 2023-03-28 at 10 11 48 AM](https://user-images.githubusercontent.com/1076924/228188616-e4a3dd51-d743-47de-9e04-6f4591b78491.png)


-   Save the configuration and then run the build by clicking on the "build now". For now this wworks manually. Select the build number and view the "console output" to see if it ran successfully.

![Screenshot 2023-03-28 at 10 18 19 AM](https://user-images.githubusercontent.com/1076924/228191497-4ac2baca-1184-4197-a528-386e66b5820a.png)


## Configuring job trigger from the github webhook

-   Configure the project/job to get triggered by the webhook whenever a change is made to the GitHub reposistory. configure the "Post-build-actions" to archive all files that result from the build. These files are called "Artifacts"

-   Make some changes to any file in your GitHub repository and oush the changes to the master branch. If configured correctly the changes should refelect on the jenkins dashboard
-   

![Screenshot 2023-03-28 at 10 55 31 AM](https://user-images.githubusercontent.com/1076924/228200106-0a7f64dd-f021-4bb4-9274-8142ed5d799a.png)


By default, artfacts are stored in this directory

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

## Configuring jenkins to copy files to the NFS via ssh

-   On the jenkis dashboard, navigate to the plugin manager, search and install "publish over ssh".

-   Configure jenkins to copy artifacts over to NFS server.
-   On the Jenkins dashboard select "Manage jenkins" and choose "Configure System" on the menu.
-   Provide the private key used for connecting to the NFS server via SSH
-   Include the Arbitrary name
-   Provide the Hostname (Private IP address of NFS server)
-   Provide the username (ec2-user, since the NFS is based on an EC2 instance running RHEL 8)
-   Provide the remote directory details (/mnt/apps since the Web servers use it as a monitoring point to retrieve files from the NFS server)

============

Test the configuration and ensure the connection returns a "sccess"

- Save the configuration. Open the Jenkins project again and go to the configuration page. Once there we will create another "posr-build action"
- Select "Send build artifacts over SSH"
- Configure it to send all files produced by the build into the previously defined remote directory. in order to do that we us ` ** `

** NB while setting this up, i ran into a permision denied issue and here is how i fixed it **

![Screenshot 2023-03-28 at 1 03 50 PM](https://user-images.githubusercontent.com/1076924/228230226-e8520130-8988-4409-9c0a-26219e3bc801.png)

In order to fix this we need to check the permission of the directory /mnt/apps on the NFS server

` ll /mnt/apps`

![Screenshot 2023-03-28 at 1 19 19 PM](https://user-images.githubusercontent.com/1076924/228233893-55e4772f-2d69-433e-96da-f084d1f9fe60.png)


This shows that the file is owned by the root user and thus we can't copy to that location. We need to change the permision of that directory by doing the following

`sudo chmod -R 777 /mnt/apps`

`sudo chown -R nonody;nobody /mnt/apps`

That should solve it. :-) 

![Screenshot 2023-03-28 at 1 30 26 PM](https://user-images.githubusercontent.com/1076924/228237275-77149927-9a70-43e2-b881-a862df9cfa4e.png)


===

![Screenshot 2023-03-28 at 1 32 20 PM](https://user-images.githubusercontent.com/1076924/228237239-67cec693-02e9-4bee-8123-98245f404d17.png)



