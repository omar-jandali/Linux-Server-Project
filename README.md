THe following is a Udacity project in which I had to setup and create a live
server that hosts a project. The purpose of the project was to show that I 
can create a secure and safe live server on the internet and figure out how to 
host a github project that is cloned onto the server. 

Linux server steps:

IP address: 34.205.90.116
Port: 2200
URL: http://34.205.90.116

login to server with the following command
- ssh ubuntu@34.205.90.116 -i  LightsailDefaultPrivateKey.pem

update all the packages for the server
- sudo apt-get update
- sudo apt-get upgrade

update the timezone data
- sudo dpkg-reconfigure tzdata
- sudo apt-get install ntp

install finger package 
- sudo apt-get finger

configure the sshd-config file
- sudo nano /etc/ssh/sshd_config
* changed port from 22 to 2200
* changed PermitRootLogin from prohibited-passwork to no
- sudo service ssh restart
* went to the network tab of the AWS server and added a new network 2200 to the available ports

Configure the FIrewall
- sudo ufw status (make sure it is disabled)
- sudo ufw default deny incoming
- sudo ufw default allow outgoing
- sudo ufw allow 2200/tcp
- sudo ufw allow 80/tcp
- sudo ufw allow 123/udp
- sudo ufw enable 
- sudo ufw status (make sure all the correct ports are open)

Downloaded a firewall monitor
- sudo apt-get install fail2ban
- sudo apt-get install sendmail
- sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
- sudo nano /etc/fail2ban/jail.local
* set the destemail field to admin users email address

Setup automatic packeges updated
- sudo apt-get install unattended-upgrades
- sudo dpkg-reconfigure --priority=low unattended-upgrades

Install apache and mod_wdgi
- sudo apt-get install apache2
- sudo apt-get install lobapache2-mod-wsgi python-dev
- sudo a2enmod wsgi
- sudo service apache2 start
* check that apache is running my going to the ip address in your address

Install git and clone project
- sudo apt-get install git
- cd /var/ww
- sudo mkdir catalog
- sudo chown -R grader:grader catalog
- cd /catalog
- git clone https://github.com/omar-jandali/Udacity-Item-Catalog.git catalog
- sudo touch catalog.wsgi
- sudo nano catalog.wsgi
**
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret_key'
**
Install all necessary packages
- sudo apt-get install python-psycopg2
- sudo apt-get install python-flask
- sudo apt-get install python-sqlalchemy
- sudo apt-get install python-pip
- sudo pip install oauth2client
- sudo pip install requests
- sudo pip install htpplib2

Change all the sqlalchemy database calls to postgresql databases
- engine = create_engine('sqlite:///restaurant.db')
- engine = create_engine('postgresql://catalog:udacity@localhost/catalog')

Create and enable Virtual Host
- sudo touch /etc/apache2/sites-available/catalog.conf
- sudo nano /etc/apache2/sites-available/catalog.conf
**
<VirtualHost *:80>
    ServerName 34.205.91.116
    ServerAdmin omarjandali@omnacore.com
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/>
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
**

Install and configure PostgreSQL
- sudo apt-get install postgresql
- sudo su - postgres
- psql
- CREATE USER catalog WITH PASSWORD 'udacity';
- ALTER USER catalog CREATEDB;
- CREATE DATABASE catalog WITH OWNER catalog;
- \c catalog;
- REVOKE ALL ON SCHEMA public FROM public;
- GRANT ALL ON SCHEMA public TO catalog;
- \q
- exit
- python database_setup.py
- python lotsofmenus.py
- python __init__.py

Create new user
- sudo adduser grader
- sudo nano /etc/sudoers.d/grader
* grader ALL=(ALL:ALL) ALL
- sudo su grader

Key based anthentication
- ssh-keygen [local machine]
- cat .ssh/udacity_key.pub [local machine]
- mkdir .ssh
- cd .ssh
- sudo touch authorized_keys
- sudo nano authorized_keys
* paste in the public key to the authorized keys file and save
- sudo chmod 700 .ssh
- sudo chmod 644 authorized_keys
- cd ..
- sudo chown -R grader:grader .ssh
Login with the new user ssh grader@34.205.91.116 -i .ssh/authorized_keys -p 2200
