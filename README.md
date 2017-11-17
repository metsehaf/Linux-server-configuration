# Linux-server-configuration
Amazon light-sail instance, Ubuntu, Apache, deploying App to a server

# Linux Server Configuration
This project was designed to teach students on how to deploy a flask App utilizing the Apache web server installed by first creating an ubuntu server instance from Amazon. We used lightsail to create a static Public ip, configure numerous modules including wsgi to assist as in deploying a Python/Flask data driven app. 
# Required for grader
### Ip address: 18.221.48.181
### ssh port: 2200
### Complete URL to hosted application: http://ec2-18-221-48-181.us-east-2.compute.amazonaws.com/

# Third party resources,Tutorials & References
###### Google:[google OAuth](https://console.developers.google.com)
###### Facebook:[Facebook OAuth](https://developers.facebook.com/)
###### Guide to deploying a Flask App on Ubuntu:[How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
###### Apache Error logs:[Where are the Apache and PHP log files?](https://askubuntu.com/questions/14763/where-are-the-apache-and-php-log-files)
###### [UFW Essentials: Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands)


# Step by Step Walkthrough
In the detailed guide below I tried to briefly document the steps I executed to get to the projects solution. The numbering and headlines should be congruent to the projects tasks 1-11. Please read through the linked sources to fully understand what is done in those steps. The walkthrough is surely far from perfect and I'm thankful for any feedback, reported errors, advice, etc. I get on this.

The parts which are not necessarily needed are marked with a *, the ones which only need to be done for extra credit ('suceeds expectations') are marked with **. I used vim as text editor in this guide. 
# 1 & 2 - Create Development Environment: Launch Virtual Machine and SSH into the server

Source: Udacity

Create new development environment.
Download private keys and write down your public IP address.
Move the private key file into the folder ~/.ssh:
$ mv ~/Downloads/udacity_key.rsa ~/.ssh/
Set file rights (only owner can write and read.):
$ chmod 600 ~/.ssh/udacity_key.rsa
SSH into the instance:
<pre>$ ssh -i ~/.ssh/udacity_key.rsa root@PUPLIC-IP-ADDRESS
# 3 & 4 - User Management: Create a new user and give user the permission to sudo

Source: DigitalOcean

Create a new user:
$ adduser NEWUSER
Give new user the permission to sudo
Open the sudo configuration:
$ visudo
Add the following line below root ALL...:
NEWUSER ALL=(ALL:ALL) ALL
*List all users (Source: Ask Ubuntu):
$ cut -d: -f1 /etc/passwd
# 5 - Update and upgrade all currently installed packages

Source: Ask Ubuntu

Update the list of available packages and their versions:
$ sudo apt-get update
Install newer vesions of packages you have:
$ sudo sudo apt-get upgrade
5** - Include cron scripts to automatically manage package updates

Source: Ubuntu documentation

Install the unattended-upgrades package:
$ sudo apt-get install unattended-upgrades
Enable the unattended-upgrades package:
$ sudo dpkg-reconfigure -plow unattended-upgrades
# 6 - Change the SSH port from 22 to 2200 and configure SSH access

Source: Ask Ubuntu

Change ssh config file:

Open the config file:
$ vim /etc/ssh/sshd_config

Change to Port 2200.

Change PermitRootLogin from without-password to no.

To get more detailed logging messasges, open /var/log/auth.log and change LogLevel from INFO to VERBOSE.
Temporalily change PasswordAuthentication from no to yes.

Append UseDNS no.

Append AllowUsers NEWUSER.
Note: All options on UNIXhelp

Restart SSH Service:
$ /etc/init.d/ssh restart or # service sshd restart

Create SSH Keys:
Source: DigitalOcean

Generate a SSH key pair on the local machine:
$ ssh-keygen

Copy the public id to the server:
$ ssh-copy-id username@remote_host -p**_PORTNUMBER_**

Login with the new user:
$ ssh -v grader@PUBLIC-IP-ADDRESS -p2200

Open SSHD config:
$ sudo vim /etc/ssh/sshd_config

Change PasswordAuthentication back from yes to no.

*Get rid of the warning message sudo: unable to resolve host ... when sudo is executed:
Source: Ask Ubuntu

Open $ vim /etc/hostname.

Copy the hostname.

Append the hostname to the first line:
$ sudo sudonano /etc/hosts

Simplify SSH login:
Logout of the SSH instance:
$ exit

Open the SSH config file on your local machine:
$ sudo vim .ssh/config

Add the following lines:
Host NEWHOSTNAME HostName PUPLIC-IP-ADDRESS Port 2200 User NEWUSER

Now, you can login into the server more quickly:
$ ssh NEWHOSTNAME

*Handle the message System restart required after login:
Source: Super User

List all packages which cause the reboot:
$ cat /var/run/reboot-required.pkgs

List everything with high security issues:
$ xargs aptitude changelog < /var/run/reboot-required.pkgs | grep urgency=high

If wanted or needed, reboot the system:
$ sudo shutdown -r now
Note: More info on rebooting on Ask Ubuntu.

# 7 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

Source: Ubuntu documentation

Turn UFW on with the default set of rules:
$ sudo ufw enable
*Check the status of UFW:
$ sudo ufw status verbose
Allow incoming TCP packets on port 2200 (SSH):
$ sudo ufw allow 2200/tcp
Allow incoming TCP packets on port 80 (HTTP):
$ sudo ufw allow 80/tcp
Allow incoming UDP packets on port 123 (NTP):
$ sudo ufw allow 123/udp
7** - Configure Firewall to monitor for repeated unsuccessful login attempts and ban attackers

Source: DigitalOcean

Install Fail2ban:
$ sudo apt-get install fail2ban
Copy the default config file:
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
Check and change the default parameters:
Open the local config file:
$ sudo vim /etc/fail2ban/jail.local
Set the following Parameters:
  set bantime  = 1800  
  destemail = YOURNAME@DOMAIN  
  action = %(action_mwl)s  
  under [ssh] change port = 2220  
Note: In the next three steps iptables is installed. However, the before installed UFW is actually a frontend for iptables and is set up already. So configuring iptables separately (as I did by just following the guide at DigitalOcean) would be a redundant step. So just install sendmail and go on with step 7.

Install needed software for our configuration:
$ sudo apt-get install sendmail iptables-persistent
Set up a basic firewall only allowing connections from the above ports:
$ sudo iptables -A INPUT -i lo -j ACCEPT
$ sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
$ sudo iptables -A INPUT -p tcp --dport 2200 -j ACCEPT
$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
$ sudo iptables -A INPUT -p udp --dport 123 -j ACCEPT
$ sudo iptables -A INPUT -j DROP
*Check the current firewall rules:
$ sudo iptables -S
Stop the service:
$ sudo service fail2ban stop
Start it again:
$ sudo service fail2ban start
# 8 - Configure the local timezone to UTC

Source: Ubuntu documentation

Open the timezone selection dialog:
$ sudo dpkg-reconfigure tzdata
Then chose 'None of the above', then UTC.
*Setup the ntp daemon ntpd for regular and improving time sync:
$ sudo apt-get install ntp
*Chose closer NTP time servers:
Open the NTP configuration file:
$ sudo vim /etc/ntp.conf
Open http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace the given servers with the new server list.
# 9 - Install and configure Apache to serve a Python mod_wsgi application

Source: Udacity

Install Apache web server:
$ sudo apt-get install apache2
Open a browser and open your public ip address, e.g. http://52.25.0.41/ - It should say 'It works!' on the top of the page.
Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
Restart the Apache server for mod_wsgi to load:
$ sudo service apache2 restart
*Get rid of the message "Could not reliably determine the servers's fully qualified domain name" after restart Source: Ask Ubuntu
Create an empty Apache config file with the hostname:
$ echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf
Enable the new config file:
$ sudo a2enconf fqdn
## 11 - Install git, clone and setup your Catalog App project

As this is by far the biggest project task, it is split in several parts.

## 11.1 - Install and configure git

Source: GitHub

Install Git:
$ sudo apt-get install git
Set your name, e.g. for the commits:
$ git config --global user.name "YOUR NAME"
Set up your email address to connect your commits to your account:
$ git config --global user.email "YOUR EMAIL ADDRESS"
## 11.2 - Setup for deploying a Flask Application on Ubuntu VPS

Source: DigitalOcean

Extend Python with additional packages that enable Apache to serve Flask applications:
$ sudo apt-get install libapache2-mod-wsgi python-dev
Enable mod_wsgi (if not already enabled):
$ sudo a2enmod wsgi
Create a Flask app:
Move to the www directory:
$ cd /var/www
Setup a directory for the app, e.g. catalog:
$ sudo mkdir catalog
$ cd catalog and $ sudo mkdir catalog
$ cd catalog and $ sudo mkdir static templates
Create the file that will contain the flask application logic:
$ sudo nano __init__.py
Paste in the following code:
python from flask import Flask app = Flask(__name__) @app.route("/") def hello(): return "Veni vidi vici!!" if __name__ == "__main__": app.run()
Install Flask
Install pip installer:
$ sudo apt-get install python-pip
Install virtualenv:
$ sudo pip install virtualenv
Set virtual environment to name 'venv':
$ sudo virtualenv venv
Enable all permissions for the new virtual environment (no sudo should be used within):
Source: Stackoverflow
$ sudo chmod -R 777 venv
Activate the virtual environment:
$ source venv/bin/activate
Install Flask inside the virtual environment:
$ pip install Flask
Run the app:
$ python __init__.py
Deactivate the environment:
$ deactivate
Configure and Enable a New Virtual Host#
Create a virtual host config file
$ sudo nano /etc/apache2/sites-available/catalog.conf
Paste in the following lines of code and change names and addresses regarding your application:
  <VirtualHost *:80>
      ServerName PUBLIC-IP-ADDRESS
      ServerAdmin admin@PUBLIC-IP-ADDRESS
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
Enable the virtual host:
$ sudo a2ensite catalog
Create the .wsgi File and Restart Apache
Create wsgi file:
$ cd /var/www/catalog and $ sudo vim catalog.wsgi
Paste in the following lines of code:
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'Add your secret key'
Restart Apache:
$ sudo service apache2 restart
## 11.3 - Clone GitHub repository and make it web inaccessible

Clone project 3 solution repository on GitHub:
$ git clone https://github.com/stueken/FSND-P3_Music-Catalog-Web-App.git
Move all content of created FSND-P3_Music-Catalog-Web-App directory to /var/www/catalog/catalog/-directory and delete the leftover empty directory.
Make the GitHub repository inaccessible:
Source: Stackoverflow
Create and open .htaccess file:
$ cd /var/www/catalog/ and $ sudo vim .htaccess
Paste in the following:
RedirectMatch 404 /\.git
## 11.4 - Install needed modules & packages

Activate virtual environment:
$ source venv/bin/activate
Install httplib2 module in venv:
$ pip install httplib2
Install requests module in venv:
$ pip install requests
*Install flask.ext.seasurf (only seems to work when installed globally):
$ *sudo pip install flask-seasurf
Install oauth2client.client:
$ sudo pip install --upgrade oauth2client
Install SQLAlchemy:
$ sudo pip install sqlalchemy
Install the Python PostgreSQL adapter psycopg:
$ sudo apt-get install python-psycopg2
# 10 - Install and configure PostgreSQL

Source: DigitalOcean (alternatively, nice short guide on Kill The Yak as well)

Install PostgreSQL:
$ sudo apt-get install postgresql postgresql-contrib
Check that no remote connections are allowed (default):
$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf
Open the database setup file:
$ sudo vim database_setup.py
Change the line starting with "engine" to (fill in a password):
python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')
Change the same line in application.py respectively
Rename application.py:
$ mv application.py __init__.py
Create needed linux user for psql:
$ sudo adduser catalog (choose a password)
Change to default user postgres:
$ sudo su - postgre
Connect to the system:
$ psql
Add postgre user with password:
Sources: Trackets Blog and Super User
Create user with LOGIN role and set a password:
# CREATE USER catalog WITH PASSWORD 'PW-FOR-DB'; (# stands for the command prompt in psql)
Allow the user to create database tables:
# ALTER USER catalog CREATEDB;
*List current roles and their attributes: # \du
Create database:
# CREATE DATABASE catalog WITH OWNER catalog;
Connect to the database catalog # \c catalog
Revoke all rights:
# REVOKE ALL ON SCHEMA public FROM public;
Grant only access to the catalog role:
# GRANT ALL ON SCHEMA public TO catalog;
Exit out of PostgreSQl and the postgres user:
# \q, then $ exit
Create postgreSQL database schema:
$ python database_setup.py
## 11.5 - Run application

Restart Apache:
$ sudo service apache2 restart
Open a browser and put in your public ip-address as url, e.g. 52.25.0.41 - if everything works, the application should come up
*If getting an internal server error, check the Apache error files:
Source: A2 Hosting
View the last 20 lines in the error log: $ sudo tail -20 /var/log/apache2/error.log
*If a file like 'g_client_secrets.json' couldn't been found:
Source: Stackoverflow
## 11.6 - Get OAuth-Logins Working

Source: Udacity and Apache

Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 52.25.0.41, its ec2-52-25-0-41.us-west-2.compute.amazonaws.com
Open the Apache configuration files for the web app: $ sudo vim /etc/apache2/sites-available/catalog.conf
Paste in the following line below ServerAdmin:
ServerAlias HOSTNAME, e.g. ec2-52-25-0-41.us-west-2.compute.amazonaws.com
Enable the virtual host:
$ sudo a2ensite catalog
To get the Google+ authorization working:
Go to the project on the Developer Console: https://console.developers.google.com/project
Navigate to APIs & auth > Credentials > Edit Settings
add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-52-25-0-41.us-west-2.compute.amazonaws.com/oauth2callback
To get the Facebook authorization working:
Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'
## 11.7** - Install Monitor application Glances

Sources: Glances and Web Host Bug

$ sudo apt-get install python-pip build-essential python-dev
$ sudo pip install Glances
$ sudo pip install PySensors
