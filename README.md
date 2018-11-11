# Udacity Linux Server Configuration
### Project Description
Secure Linux server which serves the [Udacity Catalog Project](https://github.com/mwhitman189/hotel-catalog).

The live site can be accessed at http://13.112.224.228.xip.io or http://ec2-13-112-224-228.ap-northeast-1.compute.amazonaws.com, and is hosted by AWS Amazon Lightsail.

### Objectives:
- Install a Linux distribution on a virtual machine
- Install updates
- Secure the server from attack vectors
- Install and configure web and database servers

## Server Access
IP address: `13.112.224.228.xip.io`
SSH port: `2200`
URL: http://13.112.224.228.xip.io
Connect to the server by typing the following line into the terminal in the folder with the private key file (catalogServer):
`ssh grader@13.112.224.228 -p 2200 -i .ssh/catalogServer`

## Server Setup Process
### 1. Create a new Lightsail server instance (Ubuntu 18.04) and set a static IP address
### 2. SSH into the new instance from the local terminal
### 3. Update server packages using the following:
* `sudo apt update`
* `sudo apt upgrade`
### 4. Install 'Finger' to view user info:
* `sudo apt install finger`
### 5. Change the SSH port from 22 to 2200:
* `sudo nano /etc/ssh/sshd_config`
### 6. Create new a user, 'grader', using the following:
* `sudo adduser grader`
### 7. Grant 'grader' sudo permissions by creating a new 'sudoers.d' file and changing the name in the file to 'grader'.
### 8. Generate ssh keys using ssh-keygen on the *local* machine
### 9. Copy the public key (.pub) to server using 'grader'
### 10. Force key based authentication by editing 'sshd_config':
* `sudo nano /etc/ssh/sshd_config`.
Set PasswordAuthentication to 'no', then reset ssh with:
`sudo service ssh restart`
### 11. Close all incoming Unix Firewall ports:
* `sudo ufw default deny incoming`
### 12. Open all outgoing ports:
* `sudo ufw default allow outgoing`
### 13. Open ports 22, 80, 123, 2200 in Linux:
* `sudo ufw allow [port]`
### 14. Enable UFW:
* `sudo ufw enable`
### 15. Open the same ports in the Lightsail instance through the 'Networking' tab
### 16. Set timezone to UTC:
* `sudo dpkg-reconfigure tzdata`, then select 'UTC'
### 17. Disable 'root' login:
* `sudo nano /etc/ssh/sshd_config`, then change `PermitRootLogin without-password` to `PermitRootLogin no`
### 18. Restart SSH:
* `sudo service ssh restart`
### 19. Create a Virtual Environment:
* `sudo pip install virtualenv`
* `virtualenv [VE name]`
### 20. Activate the Virtual Environment:
* `source [VE name]/bin/activate`
### 21. Install Apache2:
* `sudo apt install apache2`
### 22. Install mod_wsgi:
* `sudo apt install libapache2-mod-wsgi python-dev`
### 23. Enable mod_wsgi:
* `sudo a2enmod wsgi`
### 24. Start the server:
* `sudo service apache2 start`

## Application Setup Process
### 1. Create a project folder in '/var/www/' on the server
### 2. Clone the project into a new folder:
`git clone https://github.com/mwhitman189/hotel-catalog.git`
### 3. In the virtual environment, install modules from 'requirements.txt':
* `sudo pip install -r requirements.txt`
### 4. Create the WSGI file in the 'hotel-catalog' folder:
`#!/usr/bin/python
import sys

sys.path.insert(0, '/var/www/catalog/catalog/hotel-catalog')

from app import app as Application
application.secret_key = 'New secret key. Change it on server deployment.'`
### 5. Create a blank '__init__.py' file in '/var/www/catalog/catalog/hotel-catalog/'
### 6. Configure a new Virtual Host in '/etc/apache2/sites-available/hotel-catalog.conf':
*  `<VirtualHost *:80>
        ServerName 13.112.224.228.xip.io
        ServerAlias ec2-13-112-224-228.ap-northeast-1.compute.amazonaws.com
        ServerAdmin mwhitman189@gmail.com

        DocumentRoot /var/www/catalog/catalog/hotel-catalog

        WSGIDaemonProcess hotel-catalog user=grader group=grader processes=2 threads=25 python-path=/usr/lib/python2.7/dist-packages

        WSGIScriptAlias / /var/www/catalog/catalog/hotel-catalog/hotelCatalog.wsgi

        <Directory /var/www/catalog/catalog/hotel-catalog>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>`
### 7. Enable the virtual host:
* `sudo a2ensite hotel-catalog`

## Database Setup
### 1. Install Postgresql:
* `sudo apt install postgresql`
### 2. Login to Postgresql:
* `sudo su - postgres`
### 3. Create a new DB:
* `psql`
* `CREATE USER catalog WITH PASSWORD 'pass?word?';`
* `ALTER USER catalog CREATEDB;`
* `CREATE DATABASE catalog WITH OWNER catalog;`
### 4. Connect to the database, then logout:
* `\c catalog`
* `\q`
* `exit`
\*To login again, type: `sudo su - catalog`
### 7. Connect DB setup file to Postgresql DB by adding the following to 'models.py':
* `hostname = 'localhost'
username = 'catalog'
password = 'pass?word?
database = 'catalog'`
* `engine = create_engine('postgresql+psycopg2://' + username + ':' + password + '@localhost/' + database)`
### 6. Create Postgresql DB:
* `python models.py`

## Configure Google OAuth:
### 1. Go to https://console.cloud.google.com/ for the project and navigate to the credentials tab
### 2. Click on the edit button and add the IP address (13.112.224.228.xip.io) to 'Authorized Javascript Origins'
### 3. Add the following URIs to 'Authorized redirect URIs':
* `http://13.112.224.228.xip.io`
* `http://13.112.224.228.xip.io/oauth2callback`
* `http://13.112.224.228.xip.io/gconnect`
* `http://13.112.224.228.xip.io/login`

## Run the App
### 1. Restart Apache2:
* `sudo service apache2 restart`
### 2. Visit the web address
