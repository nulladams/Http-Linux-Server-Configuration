# Linux Server configuration - Item Catalog App deployment

The [Item Catalog App](https://github.com/nulladams/catalog) was deployed using Amazon LightSail Cloud Server. The server was configured using Linux/Ubuntu Operating System, Apache Http Server and Postgresql database.

# Item Catalog App

Item Catalog is a web application to store items according to sports categories. To create, edit and delete items, authentication and authorization are needed. When logged into the app is also possible to access an API endpoint with all categories and items registered in the database. The authentication is made through the Google OAuth service.

# Systems, Softwares and Resources

To deploy the web application, the following systems, softwares and resources were installed and configured:

- AWS LightSail Cloud Server;
- Linux/Ubuntu Operating System;
- Postgresql database;
- Google OAuth authentication service;
- Apache Http Server;
- Python3
- Flask framework;
- xip.io domain name

# Lightsail Cloud server

 First of all it was created an account on [LightSail](https://aws.amazon.com/lightsail/) and configured a server instance according to the steps bellow:
 1. Created an account
 2. Created an instance
 3. Chosen "OS Only" and then "Ubuntu 16.04 LTS"
 4. Chosen the cheapest plan available
 5. Chosen a name for the instance. It was kept the default name provided by the system
 6. In the "Networking" tab it was selected the "static IP" option
 7. In the account page, it was selected the "SSH keys" tab and then downloaded the private key
 8. It was changed permissions of the private key file
 ```sh
 $chmod 600 "private_key_file.pem"
 ```
 9. To connect to the server using the SSH client (like linux terminal)
 ```sh
 $ssh ubuntu@publicIP -i "private_key_file.pem"
 ```

# Linux Server configuration

The remote server system was configured according to the following steps:
1. Linux/Ubuntu operating system update and upgrade:
```sh
$sudo apt-get update
$sudo apt-get upgrade
```
2. Create _grader_ user:
```sh
$sudo adduser grader
```
3. Give sudo access to grader user adding to the sudoers.d directory:
```sh
$sudo nano /etc/sudoers.d/grader
```
Inside the file, add the following line
```
grader ALL=(ALL) NOPASSWD:ALL
```
4. Give to grader user ssh access to the remote server. First, generate key pair on the local machine with the command below, and name the file that will store the private key.
```sh
$ssh-keygen
```
5. On the remote server, log as grader user and create the file to store the ssh key, according to the steps below:
```sh
~$mkdir .ssh
~$touch .ssh/authorized_keys
```
6. Copy the ssh key from the local_machine_ssh_key.pub file inside the authorized_keys file, located on the remote server.
7. Set permissions to .ssh directory and authorized_keys file
```sh
~$chmod 700 .ssh
~$chmod .ssh/authorized_keys
```
8. From now on, is possible to log in the server just with the ssh key, and without the password
```sh
$ssh grader@publicIP -i ~/.ssh/local_machine_ssh_key
```
9. To disable the possibility to log with password, modify the sshd_config file, changing the option "PasswordAuthentication" to "no"
```sh
$sudo nano /etc/ssh/sshd_config
```
```
PasswordAuthentication no
```
```sh
$sudo service ssh restart
```

# Firewall and ports configuration
The firewall and ports configuration was done according to the steps bellow.

On the LightSail remote server:
1. Deny incoming requests in all ports
```sh
$sudo ufw default deny incoming
```
2. Allow outgoing responses in all ports
```sh
$sudo ufw default allow outgoing
```
3. Allow requests on port 2200
```sh
$sudo ufw allow 2200/tcp
```
4. Allow requests on port 2200
```sh
$sudo ufw allow 2200/tcp
```
5. Allow www requests
```sh
$sudo ufw allow www
```
6. Allow www requests
```sh
$sudo ufw allow www
```
7. Allow ntp requests
```sh
$sudo ufw allow ntp
```
8. Enable firewall
```sh
$sudo ufw enable
```
9. Change configuration for ssh access through 2200 port
```sh
$sudo nano /etc/ssh/sshd_config
```
```
# Port 22
Port 2200
```
```sh
$sudo service ssh restart
```

On LightSail web console:
1. Navigate to "Networking" tab
2. On Firewall, add a new Custom TCP connection on port 2200, and remove the ssh connection


# Apache and WSGI installation
Apache and mod_wsgi were installed according the steps below:

1. Apache installation
```sh
$sudo apt-get install apache2
```
2. mod_wsgi installation
```sh
$sudo apt-get install libapache2-mod-wsgi-py3
```

# Install Git and clone Item Catalog app remote repository
1. Install git
```sh
$sudo apt-get install git
```
2. Clone remote repository
```sh
$cd /var/www/
$sudo git clone https://github.com/nulladams/catalog.git catalog
```

# Python, Flask and packages installation
1. Python installation
```sh
$sudo apt-get install python3 python3-dev python3-pip
```
2. Flask and packages installation
```sh
$sudo cd /var/www/catalog
$sudo pip3 install flask
$sudo pip3 install oauth2client
$sudo pip3 install httplib2
$sudo pip3 install sqlalchemy
$sudo pip3 install requests
```

# Apache and WSGI configuration

1. Create and edit file catalog.conf
```sh
$sudo cd /etc/apache2/sites-available/
$sudo nano catalog.conf
```
```
<VirtualHost *:80>
	ServerName publicIP

	WSGIScriptAlias / /var/www/catalog/catalog.wsgi

	<Directory /var/www/catalog/>
		Require all granted
	</Directory>
	<Directory /var/www/catalog/static/>
		Require all granted
	</Directory>
</VirtualHost>
```
```sh
$sudo a2dissite 000-default.conf
$sudo a2ensite catalog.conf
```
2. Create wsgi file
```sh
$sudo cd /var/www/catalog
$sudo nano catalog.wsgi
```
```
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalogApp import app as application
application.secret_key = 'super_secret_key'
```
```sh
$sudo apache2clt restart
```


# Postgresql installation and configuration

1. Postgresql installation
```sh
$sudo apt-get install postgresql
$sudo apt-get install python3-psycopg2
```
2. Change to postgres user and access postresql:
```sh
$sudo su postgres
$psql
```
3. Create user catalog and set permissions
```sh
postgres=# CREATE USER catalog WITH PASSWORD 'catalog';
postgres=# ALTER USER catalog CREATEDB;
```
4. Create and access database catalog
 ```sh
 postgres=# CREATE DATABASE catalog WITH OWNER catalog;
 postgres=# \c catalog;
 ```
5. Deny access to catalog database for other users
```sh
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
```

# Changes on Item Catalog app python files
Some changes must be made on some files to run the application

1. Changes on google secret key path at catalogApp.py file
```
CLIENT_ID = json.loads(open('/var/www/catalog/g_client_secrets.json', 'r').read())['web']['client_id']

# on gconnect function
oauth_flow = flow_from_clientsecrets('/var/www/catalog/g_client_secrets.json',
                                            scope='')
```
2. Changes on database type and path at controller.py, models.py and loadCategories.py files
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```

# Changes on Credentials at google developers console

1. Select the "OAuth consent screen" tab
2. At "Authorized domains", add the _xip.io_ domain
3. Select the "Credentials" tab and the "OAuth client ID"
4. At "Authorized JavaScript origins", add _http://18.233.208.144.xip.io_
5. At "Authorized redirect URIs", add _http://18.233.208.144.xip.io/login_


# Loading categories into the database
Last thing is to run the loadCategories.py file to insert categories into the database
```sh
$sudo python3 loadCategories.py
```

# Project information
- IP: 18.233.208.144
- URL: 18.233.208.144.xip.io
- SSH port: 2200
- SSH key: ~/.ssh/authorized_keys

# Bibliography
- [mod_wsgi (Apache)](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/#mod-wsgi-apache)
- [SQLAlchemy](https://docs.sqlalchemy.org/en/latest/core/engines.html)
- [xip Domain name](xip.io)
- [Postgresql](https://www.postgresql.org/docs/)


# License
[MIT](https://choosealicense.com/licenses/mit/)
