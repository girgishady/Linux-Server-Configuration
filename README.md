## Linux Server Configuration project
we will take a baseline installation of a Linux server and prepare it to host our web applications. we will secure our server from a number of attack vectors, install and configure a database server, and deploy one of our existing web applications onto it.

### Files used in this project web application
* **"project.py"** python file that contain the application
* **"Database_setup.py"** python file to create the Database
* **"lotsofitems.py"** python file to create multiple categories and initial items
* **"client_secrets.json"** json file contain data for the authentication
* **"requirements.txt"** contain all application dependencies
* **"templates"** folder that contain html template files
* **"static"** folder that contain css and images


## Prerequisite
* **Ubuntu** virtual machine
* **python 2.7** 

## dependencies
The web application depend on many python modules, its included in **requirements.txt** file and you can install them using `pip install -r requirements.txt`

## Quickstart
To run the application:
* use your browser and write the following in it:
    `http://18.184.210.182.xip.io`
* **Grader** can access the server using **SSH** with port **2200** and grader **SSH key **

## steps used to setup the server as following:
1- **Get your server.**
* Start a new Ubuntu Linux server instance on [**AWS**](https://lightsail.aws.amazon.com)
- ssh to your instance.


2- **Secure your server.**
* Update all currently installed packages.
- `sudo apt-get update`
- `sudo apt-get upgrade`
* Change the SSH port from 22 to 2200
- `nano -c /etc/ssh/sshd_config` # then change the port from 22 to 2200
- `sudo service ssh restart` # restart the ssh service
* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
- `sudo ufw allow 2200/tcp`
- `sudo ufw allow 80/tcp`
- `sudo ufw allow 123/udp`
- `sudo ufw enable`

3- **Give grader access.**
*  Create a new user account named **grader**.
- `sudo adduser grader`
* Give grader the permission to sudo.
- create **grader** file in **studoers.d** folder and add user to it.
* Create an SSH key pair for grader using the **ssh-keygen** tool
- using **ssh-keygen** tool to create the key.
- install the public key in the server.
	* `mkdir .ssh`  # create a directory called ssh
	* `touch .ssh/authorized_keys`  # create a file to contain all authentication keys
	* `nano .ssh/authorized_keys`  # add the generated key to the file to be saved in the server.
	* `chmod 700 .ssh` # setting access permission
	* `chmod 644 .ssh/authorized_keys` # setting access permission 
* Disable ssh login for root user
- `sudo nano /etc/ssh/sshd_config` # Change **PermitRootLogin without-password** line to **PermitRootLogin no**
- `sudo service ssh restart` # restart the ssh service

4- **Prepare to deploy project.**
* Configure the local timezone to UTC.
- `sudo dpkg-reconfigure tzdata` # scroll to the bottom of the Continents list and select Etc or None of the above; in the second list, select UTC.
*  Install and configure Apache to serve a Python mod_wsgi application.
- `sudo apt-get install apache2` # install apache
- `sudo apt-get install libapache2-mod-wsgi python-dev` # Install mod_wsgi
- `sudo a2enmod wsgi` # Enable mod_wsg
- `sudo service apache2 start` # Start the web server with

5- **Install and configure PostgreSQL.**
* Do not allow remote connections
* Create a new database user named catalog that has limited permissions to your catalog application database.
- use the steps and configuration [here](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

6- **Install git**
-using steps in [here](https://www.liquidweb.com/kb/install-git-ubuntu-16-04-lts/)

7- **Deploy the Item Catalog project.**
* Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
- use steps [here](https://help.github.com/articles/cloning-a-repository/)

8- **Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser**
* Install virtual environment, Flask and the project's dependencies
- using the information and configuration [here](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* Configure and enable a new virtual host
- using information and configuration [here](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)
* Update OAuth authorized JavaScript origins
- update the url of the website in google authorized URI.
