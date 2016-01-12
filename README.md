# FSND Project 5 - Linux-based Server Configuration
## Purpose:
-------------------
Deploying web applications to a publicly accessible server and  secure the application

## Description:
------------------
Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

##  IP address, URL and SSH port
IP address: <http://52.34.247.81/>  
Port: 2200  
URL:<http://ec2-52-34-247-81.us-west-2.compute.amazonaws.com/>  
Username: `grader`


## Configuration
### 1 - SSH access to the Udacity Development Environment
-------------------


1. Download Private Key   
2. Move the private key file into the folder ~/.ssh  
	
		$ mv ~/Downloads/udacity_key.rsa ~/.ssh/
		
3. Change the permissions of the key, only for the owner read and write

		$ chmod 600 ~/.ssh/udacity_key.rsa
		
4. Log in the Development Environment

		$ ssh -i ~/.ssh/udacity_key.rsa root@52.34.247.81

#### Reference: 
[Udacity](https://www.udacity.com/account#!/development_environment)

### 2 - User Management & Security
-----------------

1. Create a new user name grader

		$ adduser grader

2. Grant **grader** the permission to sudo

	In the /etc/sudoers.d create a new file called grader
		
		$ nano /etc/sudoers.d/grader        
		
	In the file put in:  `grader ALL=(ALL) NOPASSWD:ALL`
	Save and qiut.
 	
3. Create a ssh key for **grader** to log in
	In your own computer terminal, generate key pairs locally
	
		$ ssh-keygen
		
	Input the key name and passphrase.  
	See the content in the public key
	
		$ cat ~/.ssh/keyname.pub
	
	Copy all the content.
	
	In the Develop Environment, change to the user **grader**
	Make a directory called **.ssh**
	
		$ mkdir ~/.ssh
		
	Create a new file called **authorized_keys** and edit it
	
		$ nano ~/.ssh/authorized_keys
	
	Paste the public key content to it, save and quit.
	
	Change the permision of the file/directory
	
		$ chmod 700 ~/.ssh
		$ chmod 644 ~/.ssh/authorized_keys
		
4. Change the SSH port from 22 to 2200, and disabled remote login of the root user 
	
	Edit configue file
		
		$ sudo nano /etc/ssh/sshd-config
		
	In hte file, change the `Port 22` to  `Port 2200`, and `PermitRootLogin without-password` to `PermitRootLogin no`(Because in the configue file `PasswordAuthentication no` is already set to disable password login, doesn't need to change it.), save and qiut.
	
	Restart ssh service
	
		$ sudo service sshd restart


#### Reference: 
[Udacity Class](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089477), [mediatemple](https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user)

### 3 - Updated application to most recent updates
---------------

1. Update package source list:

		$ sudo apt-get update

2. Upgrade all the application:

		$ sudo apt-get upgrade
		

### 4 - Include cron scripts to automatically manage package updates
----------------

1. Install unattended-upgrades package

		$ sudo apt-get install unattended-upgrades
		
2. Eable the package to automatically manage package updates

		$ sudo dpkg-reconfigure --priority=low unattended-upgrades


#### Reference: 
[Ubuntu community](https://help.ubuntu.com/community/AutomaticSecurityUpdates#Using_GNOME_Update_Manager)

### 5 - Configure the Uncomplicated Firewall(UFW)
-----------------
1. Check the ufw status

		$ sudo ufw status
2. Allow incoming connections for SSH (port 2200)

		$ sudo ufw allow 2200/tcp
	
3. Allow incoming connections for HTTP (port 80)

		$ sudo ufw allow www
	
4. Allow incoming connections for NTP (port 123)

		$ sudo ufw allow 123/udp
		
5. Eable the ufw

		$ sudo ufw enable
		
#### Reference: 
[Udacity Class](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089477)




### 6 - Configure The firewall to monitor for repeat unsuccessful login attempts and appropriately bans attackers
-----------------

1. Install the necessary package:

 		$ sudo apt-get install fail2ban
		$ sudo apt-get install sendmail iptables-persistent

2. Establish a Base Firewall

		$ sudo iptables -A INPUT -i lo -j ACCEPT
		$ sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
		$ sudo iptables -A INPUT -p tcp --dport 2200 -j ACCEPT
		$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
		$ sudo iptables -A INPUT -p udp --dport 123 -j ACCEPT
		$ sudo iptables -A INPUT -j DROP
		
3. See our current firewall rules

		$ sudo iptables -S
		
4. Adjusting the Fail2ban Configuration

		$ sudo nano /etc/fail2ban/jail.local
		
	In the file change the **bantime**: `bantime = 1800`   
	Find the **destemail**, and change it to the email address that used to collect these messages: `destemail = admin@example.com`   
	Find the `action_mwl`, change it to: `action = %(action_mwl)s`   
	Save and quit.
	
5. Restarting the Fail2ban Service

		$ sudo service fail2ban stop
		$ sudo service fail2ban start
		

#### Reference: 
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)


### 7 - Configure the local timezone to UTC
--------------------
1. Run the command below:

		$ sudo dpkg-reconfigure tzdata
		
2. Follow the screens to change the time zone.

#### Reference:
[Askubuntu](http://askubuntu.com/questions/323131/setting-timezone-from-terminal)

### 8 - Install Apache, mod_wsgi application, Git
--------------
1. Install  Apache

		$ sudo apt-get install apache2
	
	right now I can see the result on the result on  http://52.34.247.81/
	
2. Install mod_wsgi 

		$ sudo apt-get install libapache2-mod-wsgi
		
3. Install git 

		$ sudo apt-get install git
		
###9 - Configure Apache to serve a Python mod_wsgi application
-----------

1. Clone the CatalogApp on Github:

		$ cd /var/www
		$ git clone https://github.com/Nero5023/ConferenceCentral.git
		
2. To make  .git directory is not publicly accessible via a browser. Create a .htaccess file in the .git folder and put the following in this file:
		
		RedirectMatch 404 /\.git
		
	 Reference:[serverfault](serverfault)
3. Make the `project.py` file name to `__init__.py`  
4. Install pip , virtualenv (in /var/www/Catalog)
	
	
		$ sudo apt-get install python-pip 
		
	useing pip install virtualenv, and enable virtualenv
		
		$ sudo pip install virtualenv 
		
		$ sudo virtualenv venv
		$ source venv/bin/activate 
		
5. Using pip install following nessary packages
	
		$ sudo pip install Flask 
		$ sudo pip install bleach
		$ sudo pip install oauth2client
		$ sudo pip install requests
		$ sudo pip install httplib2
		$ sudo pip install redis
		$ sudo pip install passlib
		$ sudo pip install itsdangerous
		$ sudo pip install flask-httpauth
		
6. Configure and Enable a New Virtual Host

		$ sudo nano /etc/apache2/sites-available/CatalogApp.conf
	
	Add these content:
		
		<VirtualHost *:80>
				ServerName 52.34.247.81
				ServerAdmin admin@mywebsite.com
				WSGIScriptAlias / /var/www/CatalogApp/vagrant/flaskapp.wsgi
				<Directory /var/www/CatalogApp/vagrant/ItemCatalog/>
					Order allow,deny
					Allow from all
				</Directory>
				Alias /static /var/www/CatalogApp/vagrant/ItemCatalog/static
				<Directory /var/www/CatalogApp/vagrant/ItemCatalog/static/>
					Order allow,deny
					Allow from all
				</Directory>
				ErrorLog ${APACHE_LOG_DIR}/error.log
				LogLevel warn
				CustomLog ${APACHE_LOG_DIR}/access.log combined
		</VirtualHost>
				
	Eable  virtual host:
	
		$ sudo a2ensite CatalogApp
		
7. Create the .wsgi File
	 Create a file named flaskapp.wsgi with following commands:
		
		cd /var/www/CatalogApp/vagrant/
		sudo nano flaskapp.wsgi 
	
	Add these content:
	
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/CatalogApp/vagrant/")
		
		from ItemCatalog import app as application
		application.secret_key = 'Add your secret key'
		
	Right now my directory tree(In the /var/www/):
	
		CatalogApp
		--vagrant
		----flaskapp.wsgi
		----ItemCatalog
		------static
		------__init__.py
	
	
8. Some error fix:  
	In the `__init__.py` change the upload file path, backgound file path, client_secrets.json, fb_client_secrets.json to absolute path,
	
9. In Facebook developer page, Google developer console, change the applicaiton oauth path to `http://ec2-52-34-247-81.us-west-2.compute.amazonaws.com/` 

#### Reference:
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)  
[Serversforhackers](https://serversforhackers.com/configuring-apache-virtual-hosts)  
[Google Code](https://code.google.com/p/modwsgi/wiki/ApplicationIssues#Application_Working_Directory)  
[Udacity discussion](https://discussions.udacity.com/t/cant-find-image/43142/2)

###10 - Install and configure PostgreSQL
-----------

1. Install PostgreSQL
	
		$ sudo apt-get install postgresql postgresql-contrib

2. Do not allow remote connections
	in the `/etc/postgresql/9.1/main/pg_hba.conf` configure file, the default setting has already disabled remote connections
	
3. Add a new user named catalog

		$ sudo adduser catalog
4. Log into PostgreSQL
	
		$ sudo su - postgres
		$ psql
		
5. Create a new role

		CREATE ROLE catalog WITH CREATEDB;
		ALTER USER catalog WITH PASSWORD password;
	
	using `\du` can see the roles list
	
6. Create database

		CREATE DATABASE catalog WITH OWNER catalog;
	
7.  Connect to the database and lock down the permissions to only let "access_role" create tables

		\c catalog
		REVOKE ALL ON SCHEMA public FROM public;
		GRANT ALL ON SCHEMA public TO access_role;
		
	Finally press `Control`+`D` to quit 
		
#### Reference:
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)  
[Udacity discussion](https://discussions.udacity.com/t/using-postgresql-instead-of-sqlite/13575/11)


### 11 - Make the app run

1. Go to the Catalog app directory

		$ cd /var/www/CatalogApp/vagrant/

2. Edit the `__init__.py` and `database_setup.py` file:
	
	Change `engine = create_engine('sqlite:///categoryitems.db')` to `engine = create_engine('postgresql://catalog:passward@localhost/catalog')`
	
3. Restart Apache

		$ sudo service apache2 restart 
		
		
Now can visit the page thourgh <http://ec2-52-34-247-81.us-west-2.compute.amazonaws.com/>.



I learn a lot from error log (`/var/log/apache2/`)


### 12 -  Monitoring

Install Glances

		$ sudo apt-get install python-pip build-essential python-dev
		$ sudo pip install Glances
		$ sudo pip install PySensors

Type `glances` in terminal to see the result.

[Askubuntu](http://askubuntu.com/questions/293426/system-monitoring-tools-for-ubuntu)


## Third­party Resources

		unattended-upgrades
		fail2ban
		sendmail
		iptables-persistent
		apache2
		libapache2-mod-wsgi
		python-pip
		virtualenv 
		Flask 
		bleach
		oauth2client
		requests
		httplib2
		redis
		asslib
		itsdangerous
		flask-httpauth
		postgresql 
		postgresql-contrib
		Glances
		PySensors


# Nodes to Reviewer

I disable the root login, so use this key to login with user `grader`


# Thank Sonja Krause-Harder and Grant for guide me to solve my problem on udacity discussion forum.
[Sonja Krause-Harder](https://discussions.udacity.com/users/skh/activity)  
[Grant](https://discussions.udacity.com/users/gravic/activity)
