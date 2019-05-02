# FSND-Linux Server Configuration
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity. 

## Description
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

## LightSail Server Information
* IP Address: 35.182.139.148
* Grudget catalog link http://35.182.139.148/
* Accessible port: 2200
* Application URL: http://ec2-35-182-139-148.ca-central-1.compute.amazonaws.com

## 1 SSH into your server
1. Start a new Ubuntu Linux server instance on Amazon Lightsail https://lightsail.aws.amazon.com/ Download LightSail Key 
2. Move the key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	```mv ~/Downloads/somelightsailkey ~/.ssh/```
3. Open your terminal and type in
	```chmod 600 ~/.ssh/somelightsailkey```
4. In your terminal, type in
	```ssh -i ~/.ssh/somelightsailkey ubuntu@35.182.139.148```
Reference: https://stackoverflow.com/questions/46028907/how-do-i-connect-to-a-new-amazon-lightsail-instance-from-my-mac

## 2 Give grader access
1. `sudo adduser grader`
2. `nano /etc/sudoers`
3. `touch /etc/sudoers.d/grader`
4. `nano /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

## 3 Set ssh login using keys
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
	$ chown -R  grader.grader /home/grader/.ssh
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. reload SSH using `service ssh restart`
4. now you can use ssh to login with the new user you created

	`$ ssh -i linuxCourse -p 2200 grader@35.182.139.148`
	
	Your screen should looke this like this
	```
	$ ssh -i linuxCourse -p 2200 grader@35.182.139.148
	grader@35.182.139.148's password: 
	Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-1079-aws x86_64)

	* Documentation:  https://help.ubuntu.com
 	* Management:     https://landscape.canonical.com
	* Support:        https://ubuntu.com/advantage

 	Get cloud support with Ubuntu Advantage Cloud Guest:
 	http://www.ubuntu.com/business/services/cloud

	12 packages can be updated.
	0 updates are security updates.

	New release '18.04.2 LTS' available.
	Run 'do-release-upgrade' to upgrade to it.


	Last login: Thu May  2 11:02:05 2019 from 73.210.12.195
	grader@ip-172-26-6-16:~$ 
	```
	

## 4 Enforce key-based authentication
1.	$ sudo nano /etc/ssh/sshd_config. Find the PasswordAuthentication line and edit it to no.
2.	$ sudo service ssh restart.

## 5 Disable ssh login for root user
1.	$ sudo nano /etc/ssh/sshd_config. Find the PermitRootLogin line and edit it to no.
2.	$ sudo service ssh restart.

## 6 Update all currently installed packages
1. 	$ sudo apt-get update
2.	$ sudo apt-get upgrade

## 7 Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## 8 Change the SSH port from 22 to 2200
### 8.2 Modify Lightsail Network setting to enable Port 2200
1. Open the AWS panel on https://lightsail.aws.amazon.com/
2. Follow the steps to add a custom rule to enable port 2200
3. Reboot server
 Source: https://github.com/jungleBadger/-nanodegree-linux-server-troubleshoot/blob/master/Blocked_SSH_port/README.md#extra-step-to-enable-on-aws-panel

### 8.3 Change SSH port to 2200
1. Use `$ sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `$ sudo service ssh restart`

## 9 Configure the Uncomplicated Firewall (UFW)
1. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
	```
	$ sudo ufw allow 2200/tcp
	$ sudo ufw allow 80/tcp
	$ sudo ufw allow 123/udp
	$ sudo ufw enable 
	```
	
## 10 Install Apache, mod_wsgi 
1. Install Apache `$ sudo apt-get install apache2`
2. Install mod_wsgi `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Install additional libraries `$ sudo apt-get install libapache2-mod-wsgi python-dev`
3. Enable mod_wsgi `$ sudo a2enmod wsgi`
4. Restart Apache `$ sudo service apache2 restart`
	
## 11 Install git, clone your Catalog app
1. Install git
	```
	$ sudo apt-get install git
	$ cd /var/www
	$ sudo mkdir catalog
	```
	
2. Change owner of the newly created catalog folder 
	```
	$ sudo chown -R grader:grader catalog
	```
3. Clone your item catalog project
	```
	cd /catalog
	git clone https://github.com/lmidy/FSND-ItemCatalog.git catalog
	```

4. Create a catalog.wsgi file, then add this inside:

	```	
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog")

	from catalog import app as application
	application.secret_key = 'super_secret_key' 
	```

2. Rename final_project.py
`$ mv final_project.py __init__.py`

## 13 Install virtual environment
1. Install pip, so you can install python pacakages `$ sudo apt-get install python-pip` 
2. Install the virtual environment `$ sudo pip install virtualenv'
3. Move to the catalog folder: `$ cd /var/www/catalog` 
4. Create a new virtual environment with `$ sudo virtualenv venv`
5. Activate the virutal environment `$ source venv/bin/activate`
6. Change permissions `$ sudo chmod -R 777 venv

## 14 Install Flask and other dependencies
1. Install Flask `$ pip install Flask
2. Install other project dependencies 
	```
	$ sudo pip install -r requirements.txt
	```
3. sudo apt-get -qqy install postgresql python-psycopg2

## 15 Update path of client_secrets.json file
1.
2. Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json

## 16 Configure catalog virtual host and enable
1. Run this: `$ sudo nano /etc/apache2/sites-available/catalog.conf`
2. Paste this code:
```
<VirtualHost *:80>
    ServerName 35.182.139.148
    ServerAlias ec2-35-182-139-148.ca-central-1.compute.amazonaws.com
    ServerAdmin ubuntu@35.182.139.148
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
3. Enable the virtual host `$ sudo a2ensite catalog`


## 13 Install and configure PostgreSQL
1. Install PostgreSQL `$ sudo apt-get install postgresql`
2. Check if no remote connections are allowed `$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `$ sudo su - postgres`
4. Get into postgreSQL shell `$ psql`
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
9. Change create engine line in your __init__.py, database_setup.py, lotsofgrudges.py to:` engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. `$ sudo python database_setup.py`
11. `$ sudo python lotsofgrudges.py`
12. `$ sudo service apache2 restart`

## Update OAuth authorized javascript origins
1. add the URL to the authorized URI on google admin pages

## 18 Restart Apache
1.  
	```
	$ sudo service apache2 restart
	```




### References:
1. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
2. https://github.com/iliketomatoes/linux_server_configuration 
