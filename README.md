# FSND-Linux Server Configuration
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity. 

## Description
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

## LightSail Server Information
* IP Address: 35.182.177.173
* Accessible port: 2200
* Application URL: [http://ec2-34-182-177-173.us-west-2.compute.amazonaws.com/](http://ec2-34-182-177-1737.us-west-2.compute.amazonaws.com/)

## 1-Get your server
1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.
2. 

## 2-SSH into your server
1. Download LightSail Key 
2. Move the key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	```mv ~/Downloads/somelightsailkey ~/.ssh/```
3. Open your terminal and type in
	```chmod 600 ~/.ssh/somelightsailkey```
4. In your terminal, type in
	```ssh -i ~/.ssh/somelightsailkey ubuntu@35.182.177.173```
* reference: https://stackoverflow.com/questions/46028907/how-do-i-connect-to-a-new-amazon-lightsail-instance-from-my-mac

## 3-Give grader access
1. `sudo adduser grader`
2. `nano /etc/sudoers`
3. `touch /etc/sudoers.d/grader`
4. `nano /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

### 3.1 Set ssh login using keys
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key to your lightsail server

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ nano .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. reload SSH using `service ssh restart`

### 3.2 Validate and login as grader with your new access
4. now you can use ssh to login with the new user you created

	`ssh -i [privateKeyFilename] grader@35.182.177.173`

## 4 Prepare your environment
### 4.1 Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

### 4.2 Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

### 4.3 Modify Lightsail Network setting to enable Port 2200
1. Open the AWS panel on https://lightsail.aws.amazon.com/
2. Follow the steps to add a custom rule to enable port 2200
3. Reboot server
* Source: https://github.com/jungleBadger/-nanodegree-linux-server-troubleshoot/blob/master/Blocked_SSH_port/README.md#extra-step-to-enable-on-aws-panel

### 4.4 Change the SSH port from 22 to 2200
1. Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

### 4.5 Configure the Uncomplicated Firewall (UFW)
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
 
## 5. Install Package Dependencies and Item Catalog
### 5.1 Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

### 5.2 Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```

### 5.3 Clone the Catalog app from Github
1. Install git using, create catalog directory: 
``` sudo apt-get install git
	cd /var/www
	sudo mkdir catalog
```
2. Change owner of the newly created catalog folder 
```sudo chown -R grader:grader catalog
```
3. Clone your item catalog project
```cd /catalog
	git clone https://github.com/lmidy/FSND-ItemCatalog.git catalog
```

### 5.5 
1. Create a catalog.wsgi file, then add this inside:

```	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0, "/var/www/catalog/")
	from catalog import app as application
	application.secret_key = 'supersecretkey'
```

2. Rename application.py to init.py mv application.py __init__.py

### 5.6 Install virtual environment
1. Install the virtual environment sudo pip install virtualenv
2. Create a new virtual environment with sudo virtualenv venv
3. Activate the virutal environment source venv/bin/activate
4. Change permissions sudo chmod -R 777 venv

### 5.7 Install Flask and other dependencies
1. Install pip with sudo apt-get install python-pip
2.  Install Flask pip install Flask
3. Install other project dependencies sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils

### 5.8 Update path of client_secrets.json file
1. nano __init__.py
2. Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json

### 5.9 Configure and enable a new virtual host
1. Run this: sudo nano /etc/apache2/sites-available/catalog.conf
2. Paste this code:
```
<VirtualHost *:80>
    ServerName 35.167.27.204
    ServerAlias ec2-35-167-27-204.us-west-2.compute.amazonaws.com
    ServerAdmin admin@35.167.27.204
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
3. Enable the virtual host sudo a2ensite catalog

### 5.10 Install and configure PostgreSQL
* sudo apt-get install libpq-dev python-dev
* sudo apt-get install postgresql postgresql-contrib
* sudo su - postgres
* psql
* CREATE USER catalog WITH PASSWORD 'password';
* ALTER USER catalog CREATEDB;
* CREATE DATABASE catalog WITH OWNER catalog;
* \c catalog
* REVOKE ALL ON SCHEMA public FROM public;
* GRANT ALL ON SCHEMA public TO catalog;
* \q
* exit
* Change create engine line in your __init__.py and database_setup.py to: engine = create_engine('postgresql://catalog:password@localhost/catalog')
* python /var/www/catalog/catalog/database_setup.py
* Make sure no remote connections to the database are allowed. Check if the contents of this file sudo nano /etc/postgresql/9.3/main/pg_hba.conf looks like this:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

### 5.10 Restart Apache
1.  
```sudo service apache2 restart
```

### 5.11 Visit site at http://35.167.27.204

###TO REMOVE
### Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName ec2-18-216-71-137.compute-1.amazonaws.com
		ServerAdmin danielpaladar@gmail.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

### Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")
	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

### Restart Apache
1. Restart Apache `sudo service apache2 restart `

### References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
