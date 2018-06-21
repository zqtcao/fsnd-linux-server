# fsnd-linux-server
Contains server configuration information for the Udacity Linux Server Project.

The server is a Amazon Lightsail instance. To access the server via SSH, use the following:

IP Address: 52.32.29.86
User: grader
SSH Port: 2200

A live deployment of the Items Catalog project on this server can be found at the following URL:

http://ec2-52-32-29-86.us-west-2.compute.amazonaws.com/

# Walkthrough

# Part I: Lightsail Instance Setup

## Initialization
- Make sure you have an AWS account setup
- From your AWS Management Console, select "Lightsail" from under the Compute section in Services
- Create an instance using "Create Instance", select "OS Only" for the blueprint, and select "Ubuntu"
- Note that the instance is only free for the first month, so you may need to provide billing information
- Hit "Create" button and wait for the instance to be created

## Acquire a private key
- Clicking the name of the instance will take you to the instance settings page
- At the bottom of the page, click the "Account page" link to download the private key `.pem` file
- Download the key to your local machine
- SSH into the instance using `ssh ubuntu@52.32.29.86 -i your_key_name.pem`

# Part II: Initial Server Configuration

## Keep system up-to-date
- After logging in, update the packages list with `sudo apt-get update`
- Run `sudo apt-get upgrade` to upgrade installed packages

## Change SSH configuration
- Open the SSH config file using `sudo nano /etc/ssh/sshd_config`
- To change the port, modify the line `Port 22` to `Port 2200`
- To disable root login, modify the line `PermitRootLogin without-password` to `PermitRootLogin no`
- Restart the SSH service with `sudo service ssh restart`
- Login via SSH now requires the user to add an additional parameter for the port `-p 2200`

## Modify timezone to use UTC
- Use `sudo timedatectl set-timezone UTC` to change the timezone to UTC (if it isn't already)

## Update UFW (Uncomplicated Firewall) setting
- Deny all incoming conneccts on ports with `sudo ufw default deny incoming`
-

# Part III: Create account grader

## Make the account
- Switch to root user with `sudo su -`
- Create the user with `sudo adduser grader`
- Grant sudo status to user with `sudo nano /etc/sudoers.d/grader` and add the line `grader ALL=(ALL:ALL) NOPASSWD:ALL`

## Generate SSH keypair for grader
- Switch to the new grader user with `sudo su grader`
- Create the SSH directory with `sudo mkdir ~/.ssh` and add the authorized_keys file inside with `sudo touch ~/.ssh/authorized_keys`
- Change permissions for the above, `sudo chmod 700 ~/.ssh` and `sudo chmod 644 ~/.ssh/authorized_keys`
- From your local machine's terminal, create a SSH keypair using `ssh-keygen` and save the keys
- The keys should be located in `C:/Users/<yourname>/.ssh/` (Windows) as `id_rsa.pub` for the public key, `id_rsa` for the private key
- On the server's terminal, open grader's authorized_keys file with `sudo nano ~/.ssh/authorized_keys`
- Copy and paste the contents from your local `id_rsa.pub` into the authorized_keys file in the server and save
- You should now be able to login as grader `ssh grader@52.32.29.86 -p 2200` from your local machine

# Part IV: Setting up the web server

## Apache, mod_wsgi, Git
- Login as grader, begin by installing Apache `sudo apt-get install apache2`
- Install the mod_wsgi package with `sudo apt-get install libapache2-mod-wsgi`
- Install Git with `sudo apt-get install git`

## PostgreSQL and Database Setup
- Install Postgres with `sudo apt-get install postgresql`
- Switch to the postgres user and run the postgreSQL shell with `sudo su - postgres` and then `psql`
- Create the catalog database with `CREATE DATABASE catalog;`
- Create the associated user with `CREATE USER catalog;`
- Set up password with `ALTER ROLE catalog WITH PASSWORD 'password'`
- Grant "catalog" permission to the database with `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog`
- Exit the postgres account and console with `\q` followed by `exit`
- Install psycopg2 with `sudo apt-get -qqy install postgresql python-psycopg2`

## Python Packages
- install pip with `sudo apt-get install python-pip`
- install all other associated package with `sudo pip install <package_name>`. The packages used in this project are listed below:
    - `Flask`
    - `httplib2`
    - `requests`
    - `oauth2client`
    - `sqlalchemy`
    - `Flask-SQLAlchemy`

## Setup Item Catalog
- Change the directory into the Apache www folder with `cd /var/www/`
- Create a folder called FlaskApp inside with `sudo mkdir FlaskApp` and change into it with `cd FlaskApp`
- Clone the Item Catalog project into the working directory with `sudo git clone -b server https://github.com/zqtcao/fsnd-catalog.git .`
- Note that this branch setup already has `flaskapp.wsgi` setup. If you need to set up a wsgi file, use the following:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```
- Create the above file and name it `flaskapp.wsgi` and place it under `/var/www/FlaskApp/`

## Setup host
- Create the configuration file for this app with `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
- Paste the follow into the `FlaskApp.conf` file:
```
<VirtualHost *:80>
		ServerName 52.32.29.86
		ServerAdmin admin@mywebsite.com
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
- Enable the virtual host for the project with `sudo a2ensite FlaskApp.conf`

## Final preparations
- Get the host name for your particular server IP http://www.hcidata.info/host2ip.htm
- In the Google Developer Console, find the OAuth credentials for the associated application
- Add

# References
The following resources were very helpful towards completing this project.

### Udacity
- Linux Security and Web Application Server Courses
- Forums and Chat

### External Resources
- https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://www.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306
- https://github.com/hicham-alaoui/ha-linux-server-config
