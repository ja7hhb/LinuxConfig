# PROJECT DESCRIPTION
This is a project for <a href="https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004"> Udacity Full Stack Web Developer Nanodegree</a>.
Take a baseline installation of a Linux server and prepare it to host my web applications.
Secure my server from a number of attack vectors, install and configure a database server, and deployed one of my existing web application onto it.


# IP address, URL and SSH port
URL http://34.228.178.57.xip.io

IP address 34.228.178.57

SSH port 2200

You can login using this command;

```
$ ssh grader@34.228.178.57 -p 2200 -i ~/.ssh/id_rsa
```


# Step
## Create new user

Create new user "grader" in AWS lightsail terminal.
```
$ sudo adduser grader
```
Grant grader the permission to sudo
```
$ sudo nano /etc/sudoers.d/grader  
```
In the file put in: grader ALL=(ALL) NOPASSWD:ALL
```
$ sudo usermod -aG sudo grader
```

Up grade apt-get 
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get autoremove
$ sudo apt-get dist-upgrade
```
Create an SSH key pair for grader on local machine. So "id_rsa""id_rsa.pub" is created.
```
$ pwd
/Users/awakakazuyuki/.ssh
$ ssh-keygen
```
Create "authorized_keys" file and paste content of "id_rsa.pub".
```
$ sudo mkdir .ssh
$ sudo nano .ssh/authorized_keys
```

Login my AWS lightsail server from local server.
```
$ ssh grader@52.202.236.147 -p 2200 -i ~/.ssh/id_rsa
```

Change privilage of authorized_keys
```
$ sudo chmod 644 .ssh/authorized_keys
```

``` 
$ sudo nano /etc/ssh/sshd_config
``` 
Change the SSH port from 22 to 2200, and disabled login of the root user.
```
Port 22 to Port 2200
PermitRootLogin no
```
Restart Service
```
$ sudo service ssh restart
```

In my AWS lightsail page,add port range 2200 firewall settings.

Firewall settings(allow SSH port only 123, 2200, and www)
``` 
$ sudo ufw status
$ sudo ufw allow www
$ sudo ufw allow 123
$ sudo ufw allow 2200
$ sudo ufw deny 22
$ sudo ufw enable
``` 

## Configure local time zone to UTC
``` 
$ sudo dpkg-reconfigure tzdata
``` 
## Install necessary packages
Install apache2 and libapache2-mod-wsgi modules
``` 
$ sudo apt-get install apache2
``` 

Install libapache2-mod-wsgi, git and configure 
```
$ sudo apt-get install libapache2-mod-wsgi-py3
$ sudo apt-get install git
$ git config --global user.name "ja7hhb"
$ git config --global user.email "x.v.vb.ea@gmail.com"
$ sudo apt-get install postgresql postgresql-contrib
```

Clone "ItemCatalog" repo and create virtial environment, install packages.
```
$ cd /var/www
$ sudo git clone https://github.com/ja7hhb/ItemCatalog.git
$ sudo apt-get install python3-pip
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate 
$ sudo /var/www/ItemCatalog/venv/bin/pip install -r requirements.txt
```
## Configure server environment

Create the .wsgi File 
```
sudo nano catalog.wsgi
#!/usr/bin/python
import sys
import logging
import os
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/ItemCatalog/")

from catalogApp import app as application
```

Configure and Enable a New Virtual Host
```
/etc/apache2/sites-available$ sudo nano catalogApp.conf
```
Put bellow code
```
<VirtualHost *:80>
    ServerName 34.228.178.57
    ServerAlias itemcatalog.site
    WSGIScriptAlias / /var/www/ItemCatalog/catalog.wsgi
    WSGIDaemonProcess ItemCatalog python-path=/var/www/ItemCatalog/venv/lib/python3.5/site-packages

    <Directory /var/www/ItemCatalog>
        WSGIProcessGroup ItemCatalog
        WSGIApplicationGroup %{GLOBAL}
        Order deny,allow
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Some error fix about sqlite

In catalogDB.py, catalogApp.py file

ADD
```
from sqlalchemy.orm import scoped_session
session = scoped_session(sessionmaker(bind=engine))
```
Delete
```
DBSession = sessionmaker(bind=engine)
session = DBSession()
```

In catalogApp.py file line20 and line54, change 
```'client_secrets.json'``` 
=>
```'/var/www/ItemCatalog/client_secrets.json'```


Delete and recreate sqlite database
```
$ sudo rm -r catalog.db
$ python3 catalogDB.py
```

## Restart Apache to launch the app
```
sudo service apache2 restart
```

## Set up Google OAuth

Add the follwoing URL in Authorized JavaScript origins
```
http://itemcatalog.site
```
Add the folloing URL in Authorized redirect URIs
```
http://itemcatalog.site/login
http://itemcatalog.site/gconnect
http://itemcatalog.site/gdisconnect
http://itemcatalog.site/oauth2callback
```

# Third party resources
<a href="https://stackoverflow.com">StackOverFlow</a>

<a href="https://www.digitalocean.com">Digital Ocean</a>

<a href="https://askubuntu.com">Askubuntu</a>

<a href="https://github.com">github</a>
