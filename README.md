# fullstack-nanodegree-linux-server-configuration
Linux Server Configuration - Udacity Full Stack Web Developer ND

# Server details
IP address: `35.157.67.119`

SSH port: `2200`

URL: `http://ec2-35-157-67-119.eu-central-1.compute.amazonaws.com`

# Configuration changes
## Configuration of Uncomplicated Firewall (UFW)
I decided to start the project by the ufw configuration. Hence, if I got blocked out from the server, it would happen in the very beginning.

By default, block all incoming connections on all ports:

`sudo ufw default deny incoming`

Allow outgoing connection on all ports:

`sudo ufw default allow outgoing`

Allow incoming connection for SSH on port 2200:

`sudo ufw allow 2200/tcp`

Allow incoming connections for SSH on port 22 (block the connections on port 22 after the port has been changed in the sshd_config file (next step):
`sudo ufw allow 22`

Allow incoming connections for HTTP on port 80:

`sudo ufw allow www`

Allow incoming connection for NTP on port 123:

`sudo ufw allow ntp`

To check the rules that have been added before enabling the firewall use:

`sudo ufw show added`

To enable the firewall, use:

`sudo ufw enable`

To check the status of the firewall, use:

`sudo ufw status`

## Change SSH port from 22 to 2200
Edit the file `/etc/ssh/sshd_config` and change the line `Port 22` to:

`Port 2200`

## Blocking the connection to port 22
[Getting Started with UFW](https://www.howtoforge.com/tutorial/ufw-uncomplicated-firewall-on-ubuntu-15-04/)

`ufw deny 22`

To check that the rules have changed:

`sudo ufw show added`

Then restart the SSH service:

`sudo service ssh restart`
Now when the port 22 is closed I always need to SSH to my instance as a remote user.

## Update all currently installed packages

`apt-get update` - to update the package indexes

`apt-get upgrade` - to actually upgrade the installed packages

If at login the message `*** System restart required ***` is display, run the following
command to reboot the machine:

`reboot`

## Add user grader
Add user `grader` with command: `useradd -m -s grader`

## Add user grader to sudo group
Simply add the user to the sudo admin group:
```
usermod -aG sudo grader
```
Edit sudoers file:
Find the line 
`root ALL=(ALL:ALL) ALL`
and add 
`grader LL=(ALL:ALL) ALL

## Set-up SSH keys for user grader
As root user do:
```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
cp /root/.ssh/authorized_keys /home/grader/.ssh/
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```

Can now login as the `grader` user using the command:
`ssh -i ~/.ssh/YOUR_KEYFILE_NAME grader@35.157.67.119 -p 2200`

## Disable root login
Change the following line in the file `/etc/ssh/sshd_config`:

From `PermitRootLogin without-password` to `PermitRootLogin no`.

Also, uncomment the following line so it reads:
```
PasswordAuthentication no
```

Do `service ssh restart` for the changes to take effect.

Will now do all commands using the `grader` user, using `sudo` when required.

## Change timezone to UTC
Check the timezone with the `date` command. This will display the current timezone after the time.
If it's not UTC change it like this:

`sudo timedatectl set-timezone UTC`

## Install Apache to serve a Python mod_wsgi application
Install Apache:

`sudo apt-get install apache2`

Install the `libapache2-mod-wsgi` package:

`sudo apt-get install libapache2-mod-wsgi`

## Install and configure PostgreSQL
Install PostgreSQL with:

`sudo apt-get install postgresql postgresql-contrib`

To ensure that remote connections to PostgreSQL are not allowed, check
that the configuration file `/etc/postgresql/9.5cd/main/pg_hba.conf` only
allowed connections from the local host addresses `127.0.0.1` for IPv4
and `::1` for IPv6.

Create a PostgreSQL user called `catalog` with:

`sudo -u postgres createuser -P catalog`

You are prompted for a password. This creates a normal user that can't create
databases, roles (users).

Create an empty database called `catalog` with:

`sudo -u postgres createdb -O catalog catalog`

The Ubuntu documentation page on [PostgreSQL]() was helpful.

## Install packages: Flask and SQLAlchemy
Issue the following commands:

```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install --upgrade pip (not necessary)
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2 (not necessary, already installed)

```

## Install Git
`sudo apt-get install git`

## Clone the repository that contains Catalog app
Move to the `/var/www/` directory and clone the repository as the `www-data` user.
The `www-data` user will be used to run the catalog app.
```
cd :var/www/
sudo mkdir fullstack-nanodegree-vm
sudo chown www-data:www-data fullstack-nanodegree-vm/
sudo -u www-data git clone https://github.com/otsop110/fullstack-nanodegree-vm.git fullstack-nanodegree-vm
```
#### Make .git file inaccessable

```
$ sudo nano .htaccess
```

add line `RedirectMatch 404 /\.git`

[Hide Git Repos on Public Sites](https://davidegan.me/hide-git-repos-on-public-sites/)

# Modify your app structure to be ready for the deployment
[Digital Ocean tutorial on how to deploy a Flask app on an Ubuntu VPS ](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

[Patterns for flask - Larger applications](http://flask.pocoo.org/docs/0.12/patterns/packages/)

[Installing mod_wsgi on Apache server](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)

## Create a folder catalog inside the folder catalog
Move all your files in there. 
## Rename your application.py file to __init__.py.
`$ sudo mv application.py __init__.py`

## Edit engine in __init__.py, database_create.py and lotsofgenres.py
Use `sudo nano` on each of the files. Find the line
`engine = create_engine('sqlite:///filmgenrecatalog.db')`
and replace it with the following:
`engine = create_engine('postgresql://catalog:DB-PASSWORD@localhost/catalog')`

## Update the Google OAuth client secrets file
Find any reference to client_secret.json and replace it with its full path name. Also change the javascript_origins field to the IP address and AWS assigned URL of the host: "javascript_origins":["http://35.157.67.119", "http://ec2-35-157-67-119.eu-central-1.compute.amazonaws.com"]

Enter these addresses also into the Google Developers Console -> API Manager -> Credentials, in the web client under "Authorized JavaScript origins".
## Update the Facebook OAuth client secrets file
Find any reference to fb_client_secret.json and replace it with its full path name. Also change the website URL in the Facebook developers website, on the Settings page, the Site URL needs to read http://ec2-35-157-67-119.eu-central-1.compute.amazonaws.com. Then in the Products, Facebook Login, "Settings" tab, in the "Valid OAuth Redirect URIs", add http://ec2-35-157-67-119.eu-central-1.compute.amazonaws.com and http://35.157.67.119. Save these changes.
#### Create your database
```
python database_create.py
python lotsofgenres.py
```
## Create and update catalog.wsgi file for the installation
Create catalog.wsgi file in the outmost catalog folder and type in the following
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/fullstack-nanodegree-vm/vagrant/catalog/")

from catalog import app as application

application.secret_key = 'YOUR_SECRET_KEY'
```

## Configure and Enable Aapache2 to serve your app
To serve the catalog app using the Apache2 web server, you need to create a virtual host configuration file. 

`sudo nano /etc/apache2/sites-available/catalog.conf`

These are the contents:

```
<VirtualHost *:80>
	ServerName http://35.157.67.119
	ServerAdmin admin@35.157.67.119
	WSGIDaemonProcess catalog user=www-data group=www-data threads=5
	WSGIProcessGroup catalog
	WSGIApplicationGroup %{GLOBAL}
	WSGIScriptAlias / /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog.wsgi
	<Directory /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/>
		Require all granted
	</Directory>
	Alias /static /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/static
	<Directory /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/static/>
		Require all granted
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
#### Disable the default virtual host.
`sudo a2dissite 000-default.conf`
#### Enable the virtual host just created.
`sudo a2ensite catalog.conf`
#### To make these changes live restart Apache2 by
`sudo service apache2 restart`

## Run your init.py.
`python __init__.py`
#### Restart Apache 
`sudo service apache2 restart`

## Your app is now on-line!