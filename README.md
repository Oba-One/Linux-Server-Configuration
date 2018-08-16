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
# Server Configuration

#### Steps for configuration of Amazon Lightsail server using Ubuntu

***
## Connect To Server

* Connect to server using SSH `ssh -i "udacity_key.pem" ubuntu@52.11.130.21` 
2. Use Udacity Key pem 
3. Change the timezone

## Add User Grader
1. Add user **grader** with command:

sudo useradd -m -s /bin/bash grader
```

2. Set Password for the user with command:

```
sudo passwd grader
```

3. Give Sudo Permissions to Grader 

```
sudo usermod -aG sudo grader
```

4. Fix sudo resolve host error"
When the **grader** uses a sudo command, the following error appears ```sudo: unable to resolve host ip-10-20-47-177```

To fix add the hostname to the loopback address in the ```/etc/hosts``` file so the first line reads 

```127.0.0.1 localhost ip-10-20-47-177```

## Update Packages
1. Run ```sudo apt-get update``` command to update package indexes
2. Run ``` sudo apt-get upgrader``` command to upgrade and install packages

3. If at login the message **System restart required** is display, run the following command to reboot the machine:

```reboot```

## Setup SSH Keys For Grader
1. Make home directory for grader with hidden .ssh file 

```
mkdir /home/grader/.ssh
```

2. Use chown command to change the owner of the home directory to the grader.
```
chown grader:grader /home/grader/.ssh

```

3. Use chmod to change to permisions for the grader in the home directory.

```
chmod 700 /home/grader/.ssh
```

4. Copy over the ssh key from the root directory to home/grader directory 

```
cp /root/.ssh/authorized_keys /home/grader/.ssh/
```

5. Copy over the ssh key from the root directory to home/grader directory 

```
chown grader:grader /home/grader/.ssh/authorized_keys
```

5. Copy over the ssh key from the root directory to home/grader directory 

```
chmod 644 /home/grader/.ssh/authorized_keys
```

6. The grader can now login using via ssh

## Change SSH port
1. Edit the sshd_config to not permit root login, password authentication and change the port to 2200.

```
sudon nano /etc/ssh/sshd_config
```
File should chnage from:

```PermitRootLogin without-password``` to ```PermitRootLogin no```
```PasswordAuthentication no```
```Port 22``` to ```Port 2200```

3. Restart the SSH service

```
sudo service ssh restart
```
4. Now when logging in via ssh attach -p 2200 to the end

```
```


## Configure Firewall (UFW)
1. Block all incoming connections

```sudo ufw default deny incoming```

2. Allow outgoing connections

```sudo ufw default allow outgoing```

3. Allow SSH on port 2200

```sudo ufw allow 2200/tcp```

4. Allow HTTP connection on port 80

```sudo ufw allow www```

5. Allow NTP connection on port 123

```sudo ufw allow ntp```

6. Check the firewall rule

```sudo ufw show added```

7. Enable the firewall and check status

```sudo ufw enable ``` then ```sudo ufw status``

## Install Apache 
1. Install Apache to serve Python application

```sudo apt-get install apache2```

2. Install the ```libapache2-mod-wsgi``` package

```sudo apt-get install libapache2-mod-wsgi```

## Install PostgreSQL
1. Install postgresSQL with command:

`sudo apt-get install postgresql postgresql-contrib`

To ensure that remote connections to PostgreSQL are not allowed, I checked
that the configuration file `/etc/postgresql/9.5/main/pg_hba.conf` only
allowed connections from the local host addresses `127.0.0.1` for IPv4
and `::1` for IPv6.

Version number may be different from **9.5**

2. Create postgresSQL user named catalog
`sudo -u postgres createuser -P catalog`

When prompted for a password use the name grader.

3. Create an empty database called catalog
`sudo -u postgres createuser -P catalog`

## Install Flask, SQLAlchemy, & Etc
1. Run the following commands to install necessary packages:
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

## Add Application From Github
1. Install git
`sudo apt-get install git`

2. Navigate to srv directory to store project
`cd /srv`

2. Make directory for project
`sudo mkdir udacity-project`

3. Make `www-data` the owner of the directory
`sudo chown www-data:www-data udacity-project/` 

4. Clone the repo containing item catalog project.
```
sudo -u www-data git clone https://github.com/Oba-One/Item-Catalog.git udacity-project
```


## Update OAuth
1. Update Google OAuth client secret and redirect.

Fill in the `client_id` and `client_secret` fields in the file `g_client_secrets.json`.
Also change the `javascript_origins` field to the IP address and AWS assigned URL of the host.
In this instance that would be:
`"javascript_origins":["http://52.11.130.21", "http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com"]`

These addresses also need to be entered into the Google Developers Console -> API Manager
-> Credentials, in the web client under "Authorized JavaScript origins".

2. Update Facebook OAuth client secret and redirect.

In the file `fb_client_secrets.json`, fill in the `app_id` and `app_secret` fields with
the correct values.

In the Facebook developers website, on the Settings page, the website URL needs to read
`http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com`. Then in the "Advanced" tab,
in the "Client OAuth Settings" section, add `http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com`
and `http://52.11.130.21` to the "Valid OAuth redirect URIs" field. Then save these changes.

## Configure Apache 2 
1. Update catalog.wsgi file for this installation

Absolute paths are updated to where the catalog is located. The application `secret_key` is set to
something random and the PostgreSQL for the `catalog` user is set. In this case, the file
looks like this, except the password and secret is not shown for security reasons.

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/srv/udacity_project/flask/app')

from app import app as application
from catalog.database_setup import create_db
from catalog.populate_database import populate_database

application.secret_key = 'super_secret_key'  # This needs changing in production env

application.config['DATABASE_URL'] = 'postgresql://catalog:grader@localhost/catalog'
application.config['OAUTH_SECRETS_LOCATION'] = '/srv/udacity_project/flask/app/'

# Create database and populate it, if not already done so.
create_db(application.config['DATABASE_URL'])
populate_database()
```

1. Edit configuration file to allow apache to serve the application.

To serve the catalog app using the Apache web server, a virtual host configuration file
needs to be created in the directory `/etc/apache2/sites-available/`, in this case called
`catalog-app.conf`. Here are its contents:

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        ServerName 52.11.130.21

        ServerAdmin ubuntu@52.11.130.21

        # Define WSGI parameters. The daemon process runs as the www-data user.
        WSGIDaemonProcess catalog user=www-data group=www-data threads=5
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}

        # Define the location of the app's WSGI file
        WSGIScriptAlias / /srv/udacity_project/flask/app/catalog.wsgi

        # Allow Apache to serve the WSGI app from the catalog app directory
        <Directory /srv/udacity_project/flask/app/>
                Require all granted
        </Directory>

        # Setup the static directory (contains CSS, Javascript, etc.)
        Alias /static /srv/udacity_project/flask/app/static

        # Allow Apache to serve the files from the static directory
        <Directory  /srv/udacity_project/flask/app/static/>
                Require all granted
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

2. Disable the virtual default host.
`sudo a2dissite 000-default.conf`

3. Enable the catalog app virtual host.
`sudo a2ensite catalog-app.conf`

4. Restart apache.
To make these Apache2 configuration changes live, reload Apache:

`sudo service apache reload`

The catalog app should now be available at `http://52.11.130.21` and
`http://ec2-52-11-130-21.us-west-2.compute.amazonaws.com`

[This][6] was a useful guide to setting up a Flask app on Apache, though the `Directory`
permissions in the virtual host file were out of date (now it's `Require all granted`
to allow all clients to access the server).

## Extra Credit Configurations