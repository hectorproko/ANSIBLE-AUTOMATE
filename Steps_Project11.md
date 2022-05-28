# ANSIBLE AUTOMATION
The aim of this project is to start automating tasks with Ansible Configuration Management.

## INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE
I will use an already existing EC2 instance with Jenkins Installed
 

In my **GitHub** account I will create a new repository and name it **ansible-config-mgt**

Check your Ansible version by running ansible 

``` bash
sudo apt update
sudo apt install ansible
ansible --version #confirm intallation
```

Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.

Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

enterItem.pic

repositoryURL.png

Configure Webhook in GitHub and set webhook to trigger ansible build.

buildTriggers.png
addWebhook.png

Configure a Post-build job to save all (**) files, like you did it in Project 9.

Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder
``` bash  
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```

In your **ansible-config-mgt** GitHub repository, create a new branch that will be used for development of a new feature.

Checkout the newly created feature branch to your local machine and start building your code and directory structure

Create a directory and name it **playbooks** – it will be used to store all your playbook files.

Create a directory and name it **inventory** – it will be used to keep your hosts organised.

Within the playbooks folder, create your first playbook, and name it **common.yml**

Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) **dev**, **staging**, **uat**, and **prod** respectively.

InventoryPic

Created files envutally will merge to main for Jekinns to use. will either use yml or ini format, will use uat.ini for test


you need to copy your private (.pem) key to your server. Do not forget to change permissions to your private key chmod 400 key.pem, otherwise EC2 will not accept the key. Now you need to import your key into ssh-agent:
	eval `ssh-agent -s` (need this to start the agent)
ssh-add <path-to-private-key>
	Confirm the key has been added with the command below, you should see the name of your key
	ssh-add -l
	Now, ssh into your Jenkins-Ansible server using ssh-agent
	ssh -A ubuntu@public-ip
	
``` bash
ubuntu@ip-172-31-94-159:~$ eval `ssh-agent -s`
Agent pid 35692
ubuntu@ip-172-31-94-159:~$ ssh-add /home/ubuntu/daro.io.pem
Identity added: /home/ubuntu/daro.io.pem (/home/ubuntu/daro.io.pem)
ubuntu@ip-172-31-94-159:~$ ssh-add -l
2048 SHA256:wq3aSIyEU9dbylJv50YOhgXXPfcxZT7J5EmRIjLXZeA /home/ubuntu/daro.io.pem (RSA)
```


Current Files in **Main**

``` bash
hector@hector-Laptop:~/ansible-config-mgt$ git branch
* main
hector@hector-Laptop:~/ansible-config-mgt$ ls
2plays.yml  README.md
hector@hector-Laptop:~/ansible-config-mgt$
```

Current Fifles in **NewFeature**

``` bash
hector@hector-Laptop:~/ansible-config-mgt$ git branch
* NewFeature
  main
hector@hector-Laptop:~/ansible-config-mgt$ ls
inventory  playbooks  README.md
hector@hector-Laptop:~/ansible-config-mgt$
```

We do a pull request
    comparingChanges.png
    mergePull.png

We confirm
    successfullyMerged.png

``` bash
hector@hector-Laptop:~/ansible-config-mgt$ git checkout main
Switched to branch 'main'
Your branch is behind 'origin/main' by 3 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)
hector@hector-Laptop:~/ansible-config-mgt$ git branch
  NewFeature
* main
hector@hector-Laptop:~/ansible-config-mgt$ ls
2plays.yml  README.md
hector@hector-Laptop:~/ansible-config-mgt$ git pull
Warning: Permanently added the ECDSA host key for IP address '140.82.112.3' to the list of known hosts.
Updating 4fae522..84424ae
Fast-forward
 inventory/dev        | 0
 inventory/prod       | 0
 inventory/staging    | 0
 inventory/uat        | 0
 playbooks/common.yml | 0
 5 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 inventory/dev
 create mode 100644 inventory/prod
 create mode 100644 inventory/staging
 create mode 100644 inventory/uat
 create mode 100644 playbooks/common.yml
hector@hector-Laptop:~/ansible-config-mgt$ ls
2plays.yml  inventory  playbooks  README.md
hector@hector-Laptop:~/ansible-config-mgt$
```


This is supposed to trigger the build from the webhook, but is not working not just this way but overall when I tested even though it worked before
Taking this from when it worked before

    consoleOutput.png


Install plug in

Ok so in this step we are not yet using the plug in we are just testing by running the command itself
    plugin.png


So I'm testing the running of the playbook with 2plays.yml, and it works, it pings 2 instances 

``` perl
ubuntu@ip-172-31-94-159:~$ cat 2plays.yml
```
``` perl
---
- name: update web servers
  hosts: _Jenkins_Ansible
  remote_user: ubuntu

  vars_files:
    - /etc/ansible/vars/info.yml

  tasks:
    - name: Pinging
      ping:

- name: update db servers
  hosts: _NFS
  remote_user: ec2-user

  vars_files:
    - /etc/ansible/vars/info.yml

  tasks:
    - name: Pinging
      ping:
ubuntu@ip-172-31-94-159:~$
```


Ended up doing the as per instructions and run common.yml which installs wireshark on both machines. Again without the ssh agent 

``` bash
PLAY RECAP *****************************************************************************************ec2-3-220-20-204.compute-1.amazonaws.com : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ec2-54-209-253-1.compute-1.amazonaws.com : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
``` bash
Proving wireshark was installed in NFS server
[ec2-user@ip-172-31-81-201 ~]$ sudo yum list installed | grep wire
wireshark.x86_64
```

Then I changed the state: absent to remove them