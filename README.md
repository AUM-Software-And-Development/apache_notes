# apache_notes

apt update && apt upgrade
reboot (optional)

apt install tree
mkdir development
cd development
mkdir sites

from there, a decent site structure is

site_name
  site
  |-logs
  `-public
    |-media
    `-static
  src

apt install python3-pip
pip3 freeze
pip3 install virtualenv
cd into the sites directory
virtualenv venv -p python3 ---- Setup a python environment to run from in the site directory
source venv/bin/activate
pip install django
pip install pillow
pip install djangorestframework

django-admin startproject project_name .    &*Don't overlook the period decorator
vi or nano settings.py -> and add the server ip address or domain name, by changing the ALLOWED_HOSTS var to hold it
python3 manage.py runserver 0.0.0.0:8000
****************
REMINDER: USE python3 manage.py runserver 0.0.0.0:port_number
****************


MySQL:
________________

install:
  apt-install mysql-server
  mysql_secure_installation
	  disallow remote login, test databases, privileges, and anon users

goto administration commands:
mysql -u root -p
then:
CREATE DATABASE database_name
>CREATE USER 'user_name'@'localhost' IDENTIFIED BY 'password';
 GRANT ALL ON *.* TO 'user_name'@'localhost';

show databases;
SELECT User, Host FROM mysql.user;

________________
[Continuing the install to Apache after connecting to the new database]

apt install python3-dev
apt install libmysqlclient-dev
pip install mysqlclient

in the Django install find > settings.py
edit it to update the DATABASES object/dict:

'ENGINE' : 'django.db.backends.mysql',
'NAME' : 'the_db_name_on_the_localhost',
'USER' : 'user_name_without_@localhost',
'PASSWORD' : 'the_users_password',
'HOST' : 'localhost',   < (defaults to localhost if not set) Host can be a standalone servers IP, or probably domain name
'PORT' : '3306'         < This is the default. So is localhost if these two aren't set.

python3 manage.py check
python3 manage.py migrate
python3 manage.py createsuperuser

mysql
show databases;
use database_name
show tables;
select * from auth_user;
select * from auth_user \G;

python3 manage.py migrate

Confirm the server is functioning:
python3 manage.py 0.0.0.0:8000
****************
REMINDER: USE python3 manage.py runserver 0.0.0.0:port_number
****************

apt install apache2 libapache2-mod-wsgi-py3
-wsgi stands for webserver gateway interface

cd /etc/apache2/sites-available/
nano 000-default.conf
apachectl configtest

*RESTART APACHE WHEN CHANGES ARE NEEDED

service apache2 restart
service apache2 start
service apache2 stop

python3 manage.py collectstatic

Change the ownership of the available (public) directory
chown -R www-data:www-data available/

Example 000-default.conf:

<VirtualHost *:80>

	ServerName xxxx.com
	ServerAlias xxxx.me www.xxxx.com www.xxxx.me
	# Takes 1 arg, hostname and port

	# Logs:
	ErrorLog /xxxx/xxxx/xxxx/xxxx/runtime/logs/error.log
	CustomLog /xxxx/xxxx/xxxx/xxxx/runtime/logs/access.log combine

	# Alias static for Django manage.py collectstatic & visible files
	alias /static /xxxx/xxxx/xxxx/xxxx/runtime/available/static
	<Directory /xxxx/xxxx/xxxx/xxxx/runtime/available/static>
		Require all granted
	</Directory>

	# Alias media for Django manage.py collectstatic & visible files
	alias /media /xxxx/xxxx/xxxx/xxxx/runtime/available/media
	<Directory /xxxx/xxxx/xxxx/xxxx/runtime/available/media>
		Require all granted
	</Directory>

	# Location of the wsgi.py python endpoint:
	<Directory /xxxx/xxxx/xxxx/xxxx/src/project_router>	
		Require all granted
	</Directory>

	# A struct that references the Virtual Environment, as well as the manage.py path
	WSGIDaemonProcess VirtualEnvironment_symbolic_database_process python-home=/xxxx/xxxx/xxxx/xxxx/venv python-path=/xxxx/xxxx/xxxx/xxxx/src
	# Set to reference the WSGIDaemonProcess name
	WSGIProcessGroup VirtualEnvironment_symbolic_database_process
	# Decorator for the IP on port 80 as *:80/ affect ->
	WSGIScriptAlias / /xxxx/xxxx/xxxx/xxxx/src/project_router/wsgi.py

RewriteEngine on
RewriteCond %{SERVER_NAME} =xxxx.com [OR]
RewriteCond %{SERVER_NAME} =www.xxxx.com [OR]
RewriteCond %{SERVER_NAME} =xxxx.me [OR]
RewriteCond %{SERVER_NAME} =www.xxxx.me
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

<VirtualHost *:443>

	ServerName xxxx.com
	ServerAlias xxxx.me www.xxxx.com www.xxxx.me

	# Logs:
	ErrorLog /xxxx/xxxx/xxxx/xxxx/runtime/logs/error.log
	CustomLog /xxxx/xxxx/xxxx/xxxx/runtime/logs/access.log combine

	# Alias static for Django manage.py collectstatic & visible files
	alias /static /xxxx/xxxx/xxxx/xxxx/runtime/available/static
	<Directory /xxxx/xxxx/xxxx/xxxx/runtime/available/static>
		Require all granted
	</Directory>

	# Alias media for Django manage.py collectstatic & visible files
	alias /media /xxxx/xxxx/xxxx/xxxx/runtime/available/media
	<Directory /xxxx/xxxx/xxxx/xxxx/runtime/available/media>
		Require all granted
	</Directory>

	# Location of the wsgi.py python endpoint:
	<Directory /xxxx/xxxx/xxxx/xxxx/src/project_router>	
		Require all granted
	</Directory>

	# Set to reference the WSGIDaemonProcess name
	WSGIProcessGroup VirtualEnvironment_symbolic_database_process
	# Decorator for the IP on port 80 as *:80/ affect ->
	WSGIScriptAlias / /xxxx/xxxx/xxxx/xxxx/src/project_router/wsgi.py

Include /etc/letsencrypt/options-ssl-apache.conf
SSLCertificateFile /etc/letsencrypt/live/xxxx.me/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/xxxx.me/privkey.pem
</VirtualHost>

MySQL Troubleshooting:
sudo chown mysql:mysql -R /var/log/mysql
python manage.py migrate --run-syncdb

Permission Troubleshooting:
chmod -R 777 folder/
 -Reminder to change this to something more specific after

SSL:
sudo apt install certbot python3-certbot-apache

/etc/apache2/sites-available/your_domain.conf

ServerName your_domain
ServerAlias www.your_domain1 www.your_domain2 www.your_domain3 etc (splitting DNS won't secure without this)

ufw allow 'Apache Full'
ufw delete allow 'Apache'

**************** DON'T LOG OUT OF SSH WITHOUT VERIFYING THAT YOU CAN LOG BACK IN. ****************

ufw allow 'OpenSSH'

**************** DON'T LOG OUT OF SSH WITHOUT VERIFYING THAT YOU CAN LOG BACK IN. ****************

sudo certbot --apache
systemctl status certbot.timer
certbot renew --dry-run

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
