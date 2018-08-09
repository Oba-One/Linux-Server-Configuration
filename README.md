# Linux-Server-Configuration

The fifth project in Udacity's full stack web development nanodegree program.

## Project Overview

The project requires students to create a Linux Server using Amazon Lightsail. It requires hosting one of the previous completed projects with an SSH key to access the projects.  

## Server Details

* IP Address:
* SSH Port: 2200
* URL: 

## Server Configuration

### Connect To Server

1. Connect to server using SSH
2. Use Udacity Key pem 
3. Change the timezone

### Add User Grader
1. Add User for grading
2. Make password grader
3. Give Sudo Permissions to
4. Fix Sudo resolve host error 

### Update Packages
1. Run Update Command
2. Run Upgrade Command

### Setup SSH Keys For Grader
1. Make Directory for grader ssh keys
2. Use chown command
3. Use chmod command
4. Disable root login

### Change SSH port
1. Edit the sshd_config
2. Change port to 2200
3. Restart the SSH service
4. Now use command with -p 2200


### Configure Firewall (UFW)
1. Block all incoming connections
2. Allow outgoing connections
3. Allow SSH on port 2200
4. Allow HTTP connection on port 80
5. Allow NTP connection on port 123
6. Check the firewall rule
7. Enable the firewall and check status

### Install Apache 
1. Install apache with sudo
2. Install libapache package

### Install PostgreSQL
1. Install postgres 
2. Create postgres catalog user
3. Create Database called catalog

### Install Flask, SQLAlchemy, & Etc
1. Install packages with pip command

### Add Application From Github
1. Install git
2. Make directory for project
3. Clone te repo containing item catalog project


### Update OAuth
1. Update Google OAuth client secret and redirect
2. Update Facebook OAuth client secret and redirect

### Configure Apache 2 
1. Edit configuration file to allow apache to serve the application
2. Disable the virtual default host
3. Enable the catalog app virtual host
4. Restart apache

## Extra Credit Configurations