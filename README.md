# Linux-Server-Configuration
Udacity Linux Server Configuration

## Project Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## User Information
- IP address: 13.58.134.222
- Accessible SSH port: 2200
- Application URL: [Live Catalog Application](http://13.58.134.222/)

## Amazon Lightsail Set Up
- Go to [Amazon Lightsail](https://aws.amazon.com/lightsail/) website
- Create an account and login into Lightsail
- Create Lightsail instance with instance image: Ubuntu

## Server Configuration
1. Create a new user named grader and grant this user sudo permissions.
- Log into the remote VM as root user through ssh: $ ssh root@13.58.134.222
- Add a new user called grader: $ sudo adduser grader
- Create new file under the sudoers directory: $ sudo nano /etc/sudoers.d/grader. Fill that file with grader ALL=(ALL:ALL) ALL, then save
- Edit the hosts by $ sudo nano /etc/hosts, and then add 127.0.1.1 ip-10-20-37-65 under 127.0.1.1:localhost
2. Update Packages
- $ sudo apt-get update
- $ sudo apt-get upgrade
- We will also need to install FINGER, to check users status: $ apt-get install finger
3. Configure key-based Authenticatin for 'grader' user
- Open a new Terminal Window and type in $ ssh-keygen -f ~/.ssh/udacity_key.rsa
- In the same window type in $ cat ~/.ssh/udacity_key.rsa.pub to read the key, and then copy it.
- In remote VM as root user move to grader user folder by $ cd /home/grader
- Create .ssh directory with $ mkdir .ssh
- Create $ touch .ssh/authorized_keys to store the key.
- $ nano .ssh/authorized_keys and copy the udacity_key.pub key contents and save
- Change permissions: 
i. $ sudo chmod 700 /home/grader/.ssh.
ii. $ sudo chmod 644 /home/grader/.ssh/authorized_keys.
iii. Change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh
- Grader uder will now be able to log into the remote VM through ssh with: $ ssh -i ~/.ssh/udacity_key.rsa grader@13.58.134.222
4. Enforce key-based authentication
- 
