## Ansible configuration to automate tasks for project 7 to project 10

![Screenshot 2023-03-29 at 11 39 47 PM](https://user-images.githubusercontent.com/1076924/228683755-e9041a75-29f0-4ee6-aeae-e89d445c2fb5.png)

What we will accomplish 

1.  Install and configure Ansible client to act as a jump server/bastion host
2.  Create a simple Ansible playbook to automate servers configuration

====

 `sudo apt update`
 
 `sudo apt install ansible`

Create a new repository called ansible-config-mgt on github and set up webhooks on it.

https://<jenkins_url:port/github-webhooks>

On the Jenkins server, create a job called ansible and configure automatic builds when a trigger is made on the ansible-config-mgt directory via GITScm polling.

Then we test the configuration by editing the readme.md file on github.

## Preparing the environment

- Open up the terminal and connect to the Jenkins-ansible instance.
- clone the ansible-config-mgt repo and create a branch for development.

We will then do the following tasks
- create a playbook directory for storing playbooks
- Create an inventory directory for storing inventory files
- Create a common.yml file in the playbook directory
- create the following files ; dev.yml, prod.yml, staging.yml, uat.yml to represent the dev, prod, staging and uat environments.

![Screenshot 2023-04-17 at 7 41 03 PM](https://user-images.githubusercontent.com/1076924/232581017-234cf930-21d2-4015-8967-38dca9ef2d35.png)

## Setting up the inventory file

We will SSH into the target server and copy the public key into the jenkins-ansible server using the following command

```

eval 'ssh-agent -s'
ssh-add <private-key-path>

```

and then 

```
ssh-add -l

```
we will then ssh into the jenkins-ansible server using ssh-agent

`ssh -A ubuntu@public-ip-address`

=======

In the /inventory directory, we need to update the dev.yml with the configuration below

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'

```

## Creating a common playbook
Now that we have doe that we need to update the common.yml file with the configuration below

```

 ---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

## First anisble test

After updating the common.yml file we will run the following command 

`ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml`

![Screenshot 2023-04-17 at 1 19 24 PM](https://user-images.githubusercontent.com/1076924/232587284-51c92aa4-9aff-4bfd-8a8b-a230d06abdbc.png)
