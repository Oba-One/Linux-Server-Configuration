# Linux-Server-Configuration
#### The fifth project in Udacity's full stack web development nanodegree program.
***
## Project Overview

The project requires students to create a Linux Server using Amazon Lightsail. It requires hosting one of the previous completed projects with an SSH key to access the projects.  

## Server Details

* IP Address: 52.11.130.21 
* URL: http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com/
* SSH Port: 2200 


***
# Server Setup
#### Steps for configuration of Amazon Lightsail server using Ubuntu. 
***

### Connect To Server

* Connect to server with SSH with command: `ssh -i ~/.ssh/udacity_key.rsa ubuntu@52.11.130.21` 
* Run all commands needed for file permission with **sudo** in order to have admin privileges.
* Update Timezone with command: `sudo timedatectl set-timezone UTC` 


### Add User Grader

* Add user **grader** with command: `sudo useradd -m -s /bin/bash grader`
* Set Password for the user with command: `sudo passwd grader`
* Give Sudo Permissions to Grader: `sudo usermod -aG sudo grader`
* Fix sudo resolve host error:

When the **grader** uses a sudo command, the following error appears `sudo: unable to resolve host ip-10-20-47-177`

To fix add the hostname to the loopback address edit the hosts file with `sudo nano /etc/hosts` and 
change the first line to reads `127.0.0.1 localhost ip-10-20-47-177`


### Update Packages

* Run `sudo apt-get update` command to update package indexes.
* Run `sudo apt-get upgrade` command to upgrade and install packages.
* If at login the message **System restart required** is display, run the following command to reboot the machine: `reboot`.


### Setup SSH Keys For Grader

* Make home directory for grader with hidden .ssh file: `sudo mkdir /home/grader/.ssh`
* Use chown command to change the owner of the home directory to the grader: `sudo chown grader:grader /home/grader/.ssh`
* Use chmod to change to permisions for the grader in the home directory: `sudo chmod 700 /home/grader/.ssh`
* Copy over the ssh key from the ubuntu home directory to home/grader directory: `sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/`
* Change the owner to the grader: `sudo chown grader:grader /home/grader/.ssh/authorized_keys`
* With the chmod command change permissions to grader : `sudo chmod 644 /home/grader/.ssh/authorized_keys`
* The grader can now login using via ssh with command: `ssh -i ~/.ssh/udacity_key.rsa grader@52.11.130.21`  


### Change SSH Port

* Edit the sshd_config to not permit root login, password authentication and change the port to 2200: `sudo nano /etc/ssh/sshd_config`

File should change from:

* `PermitRootLogin prohibit-password` to `PermitRootLogin no`
* `PasswordAuthentication no`
* `Port 22` to `Port 2200`

* Restart the SSH service: `sudo service ssh restart`
* Now when logging in via ssh attach -p 2200 to the end of the command: `ssh -i ~/.ssh/udacity_key.rsa grader@52.11.130.21 -p 2200`


### Configure Firewall (UFW)

* Block all incoming connections: `sudo ufw default deny incoming`
* Allow outgoing connections: `sudo ufw default allow outgoing`
* Allow SSH on port 2200: `sudo ufw allow 2200/tcp`
* Allow HTTP connection on port 80: `sudo ufw allow www`
* Allow NTP connection on port 123: `sudo ufw allow ntp`
* Check the firewall rule: `sudo ufw show added`
* Enable the firewall and check status: `sudo ufw enable` then `sudo ufw status`


***
# Application Deployment
#### Application deployment using Apache 2, WSGI, Flask, and PostGresSQL.
***

### Install Apache 

* Install Apache to serve Python application: `sudo apt-get install apache2`
* Install the `libapache2-mod-wsgi` package: `sudo apt-get install libapache2-mod-wsgi`
* Now when you visit the IP address or URL you will see the default Apache start up page.


### Install PostgreSQL

* Install postgresSQL with command: `sudo apt-get install postgresql postgresql-contrib`

- To ensure that remote connections to PostgreSQL are not allowed, check
that the configuration file `/etc/postgresql/9.5/main/pg_hba.conf` only
allowed connections from the local host addresses `127.0.0.1` for IPv4
and `::1` for IPv6.

- Version number may be different from **9.5**

* Create postgresSQL user named catalog: `sudo -u postgres createuser -P catalog`

- When prompted for a password use the name password.

* Create an empty database called catalog: `sudo -u postgres createuser -P catalog`


### Install Flask, SQLAlchemy, & Etc

* Run the following commands to install necessary packages:

```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
sudo pip install flask-httpauth
sudo pip install flask-sqlalchemy
sudo pip install bleach
```

An alternative to installing system-wide python modules is to create a virtual
environment for each application using the [virualenv][4] package.


### Add Application  From Github

* Install git: `sudo apt-get install git`
* Navigate to www directory to store project: `cd /var/www`
* Clone the repo containing item catalog project: `sudo -u www-data git clone
https://github.com/Oba-One/Linux-Application.git FlaskApp`


### Update OAuth

* Update Google OAuth client secret and redirect"

Fill in the `client_id` and `client_secret` fields in the file `g_client_secrets.json`.
Also change the `javascript_origins` field to the IP address and AWS assigned URL of the host.
In this instance that would be:

`"javascript_origins":["http://52.11.130.21",
"http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com"]`

`"authorized-redirects":["http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com/gconnect"
"http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com/login"]`

These addresses also need to be entered into the Google Developers Console -> API Manager
-> Credentials, in the web client under "Authorized JavaScript origins".

* Update Facebook OAuth client secret and redirect:

In the file `fb_client_secrets.json`, fill in the `app_id` and `app_secret` fields with
the correct values.

In the Facebook developers website, on the Settings page, the website URL needs to read
`http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com`. Then in the "Advanced" tab,
in the "Client OAuth Settings" section, add `http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com`
and `http://52.11.130.21` to the "Valid OAuth redirect URIs" field. Then save these changes.

- Facebook no longer allows http authentication, all domains must have https.


### Configure Apache 2 

* Navigate to the apache2/sites-available directory: `cd /etc/apache2/sites-available/`
* Apache requires a `.conf` file to serve application. The file needed is located on github. 
Clone the directory: `git clone https://github.com/Oba-One/Linux-Server-Configuration.git hi`
* Cd into the hi folder and move the `flask-app.conf` file to `/sites-available/`: 
`cd hi` then  `sudo mv flask-app.conf /etc/apache2/sites-available/
* Delete the `hi` directory: `sudo rm -r hi`
* If any files chages need to be made run: `sudo nano flask-app.conf` to edit file.

Here's what the file should contain:

```python
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. 
        ServerName 52.11.130.21
        ServerAdmin hello@yoyo.com
        # Define WSGI parameters. The daemon process runs as the www-data user.
        WSGIDaemonProcess catalog user=www-data group=www-data threads=5
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}
        # Define the location of the app's WSGI file
        WSGIScriptAlias / /var/www/FlaskApp/catalog/catalog.wsgi
        # Allow Apache to serve the WSGI app from the catalog app directory
        <Directory /var/www/FlaskApp/catalog/>
                Require all granted
        </Directory>
        # Setup the static directory (contains CSS, Javascript, etc.)
        Alias /static /var/www/FlaskApp/catalog/static
        # Allow Apache to serve the files from the static directory
        <Directory  /var/www/FlaskApp/catalog/static/>
                Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Navigate to the /var/www/FlaskApp/catalog directory to view and edit the `catalog.wsgi file.`

- This file is use to connect your application to apache and allow it to be deployed.

* Use to command: `sudo nano catalog.wsgi` to view and edit the file. 

- It's already configured but if changes need to be made edit this file.

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/FlaskApp/catalog/')

from __init__.py import app as application

application.secret_key = 'super_secret_key'  # This needs changing in production env
```

- Absolute paths are updated to where the catalog is located. The application `secret_key` is set to
something random and the PostgreSQL for the `catalog` user is set. In this case, the file
looks like this, except the password and secret is not shown for security reasons.

* On order to now view the application Apache has to switch from serving the default conf file to the new one.
* Disable the virtual default host: `sudo a2dissite 000-default.conf`
* Enable the catalog app virtual host: `sudo a2ensite flask-app.conf`
* Reload Apache for changes to take effect: `sudo service apache2 reload`

* The catalog app should now be available at `http://52.11.130.21` and
`http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com`

* If any errors occur navigate to the `/var/log/apache2/` and `sudo nano error.log` to view errors.
***

# Third Party Resources
#### Comprehensive list of third party resources that helped with the fustrations and headaches that come along with deploying your first app.  
***

* [Installing Apache](https://www.digitalocean.com/community/tutorials/apache-basics-installation-and-configuration-troubleshooting) 
* [Deploying Flask App on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Setting Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604)
* [Hello Flask with Apache WSGI](http://www.bogotobogo.com/python/Flask/Python_Flask_HelloWorld_App_with_Apache_WSGI_Ubuntu14.php)
* [Installing Mod WSGI](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)