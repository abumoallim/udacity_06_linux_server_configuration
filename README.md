# Linux Server Configuration

### Project Overview
> In this project, you'll work with publishing your flask website with aws using ubuntu server.

### Steps
- Create  Amazon Lightsail instance
	- https://aws.amazon.com/lightsail/

- My Amazon Instance info:
	- Domain name : `http://ec2-13-233-27-9.ap-south-1.compute.amazonaws.com/`
	- Public IP : `13.233.27.9`

- Update all the currently installed Packages
	- `sudo apt-get update` (Will update all the references to application on ubuntu and it is stored in /etc/apt/sources.list)
        - `sudo apt-get upgrade`(Will upgrade all the applications)

- Change SSH default port to 2200
  	- Go to `cd etc/ssh/sshd_config`
	- Change the line : `Port 22 to Port 2200`

- Configuring Firewall
	- Check your firewall status as : `sudo ufw status`
	- Deny all incoming request : `sudo ufw default deny incoming`
	- Allow all outgoing request : `sudo ufw default allow outgoing`
	- Now allow ssh : `sudo ufw allow ssh`
	- But ssh is on port 2200 : `sudo ufw allow 2200/tcp`
	- Allow all Http request : `sudo ufw allow www`
	- Allow all NTP request : `sudo ufw allow ntp`
	- Enable firewall : `sudo ufw enable`

- User Management
	- Creating user : `sudo adduser USER_NAME`
	- Connect to that user : `ssh USER_NAME@127.0.0.1 -p 2200`
	- Check : in `/etc/ssh/sshd_config` having `PasswordAuthentication no`
	- than it will ask for publickey authorization else if it yes than you can login with password

- Now after Changing port to 2200, amazon lightsail wont allow you to get into its console because it will port to 22 but you have to changed it to 2200
	- Go to `https://lightsail.aws.amazon.com/ls/webapp/ap-south-1/instances/YOUR-INSTANCE/networking` and `add custom port 2200`
	- Go to `https://lightsail.aws.amazon.com/ls/webapp/account/keys` and download default keys
	- Change the permission for that file : `chown 400 -path-to-downloaded-file`
	- Connect to lightsail ubuntu(root) instance : `ssh -i -path-to-your-public-key username@public_ip -p 2200`
	- Now you can connect to lightsail instance from your computer's ssh

- Make our new user sudoer (Allow them to run sudo commands)
	- In this file : `sudo cat /etc/sudoers` (This fill contains all the permission for root but do not edit in this file because this file may change if system 		updates) instead see last line of `sudoers.d`
	- check directory : `sudo ls /etc/sudoers.d`
	- add file in `/etc/sudoers.d` with name of user you created and add line to that file: `grader ALL=(ALL) NOPASSWD:ALL and save the file`

- Connect to user with PublicKey authentication:
	- Generate key on your local machine rather than our ubuntu server image
	- Use keygen : `ssh-keygen`
	- Give path and name: /root/linuxCourse (I have given linuxCourse file name to this public key)
	- it will result in :
		our identification has been saved in /root/.ssh/linuxCourse.
		Your public key has been saved in /root/.ssh/linuxCourse.pub.
		Now you need to upload this public key to our ubuntu server
	- On server login in to our grader(another one) user first with password
	- `mkdir .ssh`
	- `touch .ssh/authorized_keys`
	- Go to our local machine and get that key we generated from keyGen : `sudo cat /root/linuxCourse.pub`
	- copy all the lines 
	- Go to server : `nano .ssh/authorized_keys` and paste and save
	- change permission for that file on server : `chmod 700 .ssh`
	- `chmod 644 .ssh/authorized_keys`
	
- Connecting user:
	- For grader user : `sudo ssh grader@13.233.27.9 -p 2200 -i /root/.ssh/linuxCourse`
	- For Ubuntu(root) user : `sudo ssh ubuntu@13.233.27.9 -p 2200 -i path-to-file-on-local-machine/file-name.pem`
	

- Deploying the Project:
	- First install apache : `sudo apt-get install apache2`
	- Install mod_wsgi_package : `sudo apt-get install libapache2-mod-wsgi-py3`
	- Git clone your project into `/var/www/`

- Creating virtual server:
	- `pip3 install virtualenv`
	- `python3 -m pip install virtualenv` (or `sudo python3 -m virtualenv -p /usr/bin/python3 .venv` )
	- `source .venv/bin/activate`
	
- Installing database:
	- `sudo apt-get install -y postgresql postgresql-contrib`

- Connecitng to database:
	- Connect to psql: `sudo -u postgres psql`
	- Create Role :     `postgres=> CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog'; (do not forget the semi colon)`
    	- Create Database : `postgres=> CREATE DATABASE catalog;`
    	- Grant Privileges: `postgres=> GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
    	- exit 		 : `postgres=> \q`
 	- Switching your currnet database to postgress:
		- \c postgres (connecting to postgres database)
		- \c catalog (connectiong to our database)
	- Flask Migrations
		- `Export FLASK_APP = application.py`
		- flask db init
		- flask db migrate
		- flask db upgrade

- Checking if all libraries are already installed
	- `pip install -r requirements.txt` (if you dont have requirements.txt use pipreqs path_to_your_project/)

- Setting up the project (In my case Catalog_item_application : https://github.com/abumoallim/udacity_04_catalog_item_application)
	- Change google sign in redirect_uri from localhost to domain of aws (you can find your domain name using reverse ip)
	- Update your authorize redirect url in `https://console.cloud.google.com/apis/credentials` 
	- change `SQLALCHEMY_DATABASE_URI: "postgresql://catalog:catalog@localhost/catalog"`

- Updating apache config and wsgi file :
	- Now you need to add wsgi file inside our project as : catalog_item_application.wsgi
	- create a new config file at `/etc/apache2/sites-available/catalog_item_application.conf`
	- delete default 0000-default.config
	- Enable the site: `sudo a2ensite catalog_item_application`
	- Reload Apache: `sudo service apache2 reload`
  	
- Checking erro Logs:
	- go to : /var/logs/apache/error.log
	
