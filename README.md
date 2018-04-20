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
### 1. Create a new user named grader and grant this user sudo permissions.
- Log into the remote VM as root user through ssh: $ ssh root@13.58.134.222
- Add a new user called grader: $ sudo adduser grader
- Create new file under the sudoers directory: $ sudo nano /etc/sudoers.d/grader. Fill that file with grader ALL=(ALL:ALL) ALL, then save
- Edit the hosts by $ sudo nano /etc/hosts, and then add 127.0.1.1 ip-10-20-37-65 under 127.0.1.1:localhost
### 2. Update Packages
- $ sudo apt-get update
- $ sudo apt-get upgrade
- We will also need to install FINGER, to check users status: $ apt-get install finger
### 3. Configure ey-based Authenticatin for 'grader' user
- Open a new Terminal Window and type in $ ssh-keygen -f ~/.ssh/udacity_key.rsa
- In the same window type in $ cat ~/.ssh/udacity_key.rsa.pub to read the key, and then copy it.
- In remote VM as root user move to grader user folder by $ cd /home/grader
- Create .ssh directory with $ mkdir .ssh
- Create $ touch .ssh/authorized_keys to store the key.
- $ nano .ssh/authorized_keys and copy the udacity_key.pub key contents and save
- Change permissions: 
1. $ sudo chmod 700 /home/grader/.ssh.
2. $ sudo chmod 644 /home/grader/.ssh/authorized_keys.
3. Change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh
- Grader user will now be able to log into the remote VM through ssh with: $ ssh -i ~/.ssh/udacity_key.rsa grader@13.58.134.222
### 4. Enforce key-based Authentication
- $ sudo nano /etc/ssh/sshd_config. Find the PasswordAuthentication line and edit it to no
- Change the SSH port from 22 to 2200
1. $ sudo nano /etc/ssh/sshd_config. Find the Port line and edit it to 2200
2. Disable ssh login for root user: $ sudo nano /etc/ssh/sshd_config. Find the PermitRootLogin line and edit it to no
3. $ sudo service ssh restart
- Grader user will now be able to log into the remote VM through ssh with: $ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.58.134.222
### 5. Configure UFW
- $ sudo ufw allow 2200/tcp
- $ sudo ufw allow 80/tcp
- $ sudo ufw allow 123/udp
- $ sudo ufw enable
## Catalog Application Deployment
### 1. Install Required Packages
- $ sudo apt-get install apache2
- $ sudo apt-get install libapache2-mod-wsgi python-dev
- $ sudo apt-get install git
### 2. Enable mod_wsgi
- $ sudo a2enmod wsgi
- start the web server by $ sudo service apache2 start
### 3. Set Up Directory and Folder for Catalog Application
- $ cd /var/www
- $ sudo mkdir catalog
- $ sudo chown -R grader:grader catalog
- $ cd catalog
### 4. Clone Catalog Project from Github to catalog folder
- $ git clone https://github.com/iperun/Catalog catalog
### 5. Create catalog.wsgi file to serve the application
- $sudo nano catalog.wsgi
- Input the following:
```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```
### 6. Rename the project.py to `__init__.py`
### 7. Install Envirnment, Flask, and Project Dependencies
- $ sudo apt-get install python-pip
- $ sudo pip install virtualenv
- $ sudo virtualenv venv
- $ source venv/bin/activate
- $ sudo chmod -R 777 venv
- $ sudo pip install Flask
- $ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 requests 
### 8. Configure and Enable New Virtual Host
- Create a virtual host conifg file: $ sudo nano /etc/apache2/sites-available/catalog.conf
- Input the following: 
```python
<VirtualHost *:80>
    ServerName 13.58.134.222
    ServerAlias ec2-13-58-134-222.us-east-2.compute.amazonaws.com
    ServerAdmin admin@13.58.134.222
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Enable the new virtual host: $ sudo a2ensite catalog
### 9. Install PostgreSQL and Set Up Database
- $ sudo apt-get install libpq-dev python-dev
- $ sudo apt-get install postgresql postgresql-contrib
- $ sudo su - postgres
- Connect to database with $ psql
### 10. Create user and Set Up database
- Create new user called 'catalog': # CREATE USER catalog WITH PASSWORD 'password';
- Give catalog user the CREATEDB capability: # ALTER USER catalog CREATEDB;
- $ CREATE DATABASE catalog WITH OWNER catalog;
- Connect to database $ \c catalog
- Revoke all rights: # REVOKE ALL ON SCHEMA public FROM public;
- Permissions to only let catalog role to create: # GRANT ALL ON SCHEMA public TO catalog;
- Log out from PostgreSQL: # \q
- Return to the grader user: $ exit
### 11. Change Engine in Application
- In `__init__.py`, database_setup.py, and catalog_items.py files change database engine to engine = create_engine('postgresql://catalog:password@localhost/catalog')
### 12. Initiate database_setup.py and catalog_items scripts
- python database_setup.py
- python catalog_items.py
### 13. Restart the Apache Server and the Application is Online Now
## Reference
- http://www.hcidata.info/host2ip.cgi
- https://github.com/callforsky/udacity-linux-configuration
- https://github.com/mulligan121/Udacity-Linux-Configuration/blob/master/README.md
- https://ubuntuforums.org/forumdisplay.php?f=339&s=729643b53c419b901c12df9b9e9a2af0
- https://www.youtube.com/watch?v=HcwK8IWc-a8

