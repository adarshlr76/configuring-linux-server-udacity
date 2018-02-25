# configuring-linux-server-udacity

udacity-linux-server-configuration
Project Description

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

    IP address: 13.127.29.53

    Accessible SSH port: 2200

    Application URL: http://ip-172-26-14-133.ap-south-1a.compute.amazonaws.com/

    

Walkthrough

    Create new user named grader and give it the permission to sudo

    SSH into the server through ssh -i ~/.ssh/udacity_key.pem ubuntu@13.127.29.53
    Run $ sudo adduser grader to create a new user named grader
    Create a new file in the sudoers directory with sudo vim /etc/sudoers.d/grader
    Add the following text grader ALL=(ALL:ALL) ALL
    Run sudo nano /etc/hosts
    Prevent the error sudo: unable to resolve host by adding this line 127.0.1.1 ip-172-26-14-133

    Update all currently installed packages

    Download package lists with sudo apt-get update
    Fetch new versions of packages with sudo apt-get upgrade

    Change SSH port from 22 to 2200

    Run sudo vim /etc/ssh/sshd_config
    Change the port from 22 to 2200
    Make sure the new port is added to the INBOUND rules for your security group. change firewall on networking tab to custom tcp 2200
    Confirm by running ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@13.127.29.53
    

    Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/udp
    sudo ufw enable

    Configure the local timezone to UTC

    Run sudo dpkg-reconfigure tzdata and then choose UTC

    Configure key-based authentication for grader user
    Run this command cp /ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys

    Disable ssh login for root user

    Run sudo vim /etc/ssh/sshd_config
    Change PermitRootLogin  line to PermitRootLogin no
    Restart ssh with sudo service ssh restart
    Now you are only able to login using ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.127.29.53

    Install Apache

    sudo apt-get install apache2

    Install mod_wsgi

    Run sudo apt-get install libapache2-mod-wsgi python-dev
    Enable mod_wsgi with sudo a2enmod wsgi
    Start the web server with sudo service apache2 start

    Clone the Catalog app from Github

    Install git using: sudo apt-get install git
    cd /var/www
    sudo mkdir catalog
    Change owner of the newly created catalog folder sudo chown -R grader:grader catalog
    cd catalog
Clone your project from github git clone https://github.com/adarshlr76/item-catalog

    sudo mkdir ~/catalog
    cd ~
    sudo git clone https://github.com/adarshlr76/item-catalog catalog
    sudo ln -sT ~/catalog/vagrant/catalog /var/www/catalog

    Install Flask and other dependencies using pip tool (which also needs to be installed)

Install pip with sudo apt-get install python-pip
Install Flask pip install Flask
Install other project dependencies sudo pip install httplib2 oauth2client sqlalchemy psycopg2-binary sqlalchemy_utils requests
    install flask
sudo -H pip install flask



    Create a catalog.wsgi file, then add this inside:

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog")

from catalog import app as application
application.secret_key = 'supersecretkey'


    Rename application.py to catalog.py mv application.py __init__.py

    sudo vi __init__.py
    change app.run(host = '0.0.0.0',8000) TO
    app.run()

    update path of client_secrets.json and fb_client_secrets.json
    sudo vi __init__.py
    /var/www/catalog/client_secrets.json

    and 
    /var/www/catalog/fb_client_secrets.json


    Configure and enable a new virtual host

    Run this: sudo vi /etc/apache2/sites-available/catalog.conf
    Paste this code:

<VirtualHost *:80>


    WSGIDaemonProcess catalog python-path=/var/www/catalog/
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    ServerName 13.127.29.53
    ServerAdmin webmaster@13.127.29.53
    DocumentRoot /var/www/catalog
</VirtualHost>

    Install and configure PostgreSQL

    sudo apt-get install libpq-dev python-dev
    sudo apt-get install postgresql postgresql-contrib
    sudo su - postgres
    psql
    CREATE USER catalog WITH PASSWORD 'password';
    ALTER USER catalog CREATEDB;
    CREATE DATABASE catalog WITH OWNER catalog;
    \c catalog
    REVOKE ALL ON SCHEMA public FROM public;
    GRANT ALL ON SCHEMA public TO catalog;
    \q
    exit




Change create engine line in your __init__.py , add_catalog.py and database_setup.py to: engine = create_engine('postgresql://catalog:password@localhost/catalog')

    run the below commands
python database_setup.py
python add_catalog.py

   Restart Apache

    sudo service apache2 restart

    Visit site at http://13.127.29.53

if something isn't working, try checking the log file in /var/log/apache2/error.log).
   

    Install virtual environment

    Install the virtual environment sudo pip install virtualenv
    Create a new virtual environment with sudo virtualenv venv
    Activate the virutal environment source venv/bin/activate
    Change permissions sudo chmod -R 777 venv


  
    Enable the virtual host sudo a2ensite catalog

    
