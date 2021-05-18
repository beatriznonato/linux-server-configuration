# Linux Server Configuration :computer: 

![pep8](https://img.shields.io/badge/pep8online-compliant-green.svg)

## Description of Project

Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com) and prepare it to host web applications,
including installation updates, providing security against multiple attack vectors, and installing 
Web and Database servers.
The reference installation will be the [Item Catalog Project](https://github.com/beatriznonato/itemCatalog), where you can log in with your [Gmail](https://mail.google.com) 
account allowing you to create, read, update and delete files within the app. 

## Instructions for SSH access to the instance

1. Download Private Key below
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	```mv ~/Downloads/udacity_key.rsa ~/.ssh/```
3. Open your terminal and type in
	```chmod 600 ~/.ssh/udacity_key.rsa```
4. In your terminal, type in
	```ssh -i ~/.ssh/udacity_key.rsa root@52.24.125.52```
5. Development Environment Information

	Public IP Address

	 107.22.135.163
	
	:closed_lock_with_key: Private Key (is not provided here).
  
  
## Create a New user named grader
  
1. `sudo adduser grader`
2. `vim /etc/sudoers`
3. `touch /etc/sudoers.d/grader`
4. `vim /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`
5. `:wq` save and quit.

## Set ssh login using keys

1. Generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. Deploy public key on developement enviroment

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vim .ssh/authorized_keys
	```
	Copy the Public Key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. Reload SSH using `service ssh restart`
4. Now you can use ssh to login with the new user you created

	`ssh -i [privateKeyFilename] grader@107.22.135.163`
  
## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade
  
## Change the SSH Port from 22 to 2200

1. Use `sudo vim /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , in terminal use command `:wq` for save & quit.
2. Reload SSH using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable
  
## Configure the Local Timezone to UTC

1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## Install and Configure PostgreSQL

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
  
## Install git, clone and setup your Catalog App project.

1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/beatriznonato/itemCatalog`
6. Rename the project's name `sudo mv ./itemCatalog ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `project.py` to `__init__.py` using `sudo mv project.py __init__.py`
9. Edit `project.py`, `db_setup.py` and `create_db_items.py` and change `engine = create_engine('sqlite:///catalog.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Use pip to install dependencies `sudo pip install -r requirements.txt`
12. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
13. Create database schema `sudo python database_setup.py`
 
## Configure and Enable a New Virtual Host

1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 107.22.135.163
		ServerAdmin bia_nonato@icloud.com
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

## Create the .wsgi File

1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo vim flaskapp.wsgi 
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

## Restart Apache

Restart Apache `sudo service apache2 restart `

## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

### License
MIT © Beatriz Nonato :woman_technologist:
