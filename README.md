# Udacity Full Stack Web Developer Nanodegree

## Linux Server Configuration
> Armando Perez

### IP and URL:
> Public IP: 34.219.79.240
> Host name: ec2-34-219-79-240.us-west-2.compute.amazonaws.com


### How To:  
#### Amazon Lightsail
* Create Lightsail account and new Instance
* Connect using SSH
* Download private key
* In the Networking tab, add two new custom ports - 123 and 2200
#### Server configuration
* Place private key in .ssh
* `$ chmod 600 ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem`
* `$ ssh -i ~/.ssh/LightsailDefaultKey-us-west-2.pem ubuntu@34.219.79.240`
#### Create new account grader
* `$ sudo su -`
* `$ sudo nano /etc/sudoers.d/grader`
    Add `grader ALL=(ALL:ALL) ALL`
* `$ sudo nano /etc/hosts`
    Under `127.0.1.1:localhost` add `127.0.1.1 ip-10-20-37-65`
#### Install updates and finger package
* `$ sudo apt-get update`
    `$ sudo apt-get upgrade`
    `$ sudo apt-get install finger`
#### Keygen
* In a new terminal, `$ ssh-keygen -f ~/.ssh/udacity_key.rsa
* `$ cat ~/.ssh/udacity_key.rsa.pub`
* In the original terminal, `$ cd /home/grader`
* `$ mkdir .ssh`
* `$ touch .ssh/authorized_keys`
* `$ nano .ssh/authorized_keys`
* Permissions:
    `$ sudo chmod 700 /home/grader/.ssh`
    `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`
* `$ sudo chown -R grader:grader /home/grader/.ssh`
* `$ sudo service ssh restart`
* To disconnect:
    `$ ~.`
* `$ ssh -i ~/.ssh/udacity_key.rsa grader@18.221.145.45`
#### Enforce key based authentication
* `$ sudo nano /etc/ssh/sshd_config`
* Find the PasswordAuthentication line and change text after to `no`
* `$ sudo service ssh restart`
#### Change port
* `$ sudo nano /etc/ssh/sshd_config`
* Find the Port line and change `22` to `2200`
* `$ sudo service ssh restart`
* `$ ~.`
* `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@34.219.79.240`
#### Disable root login
* `$ sudo nano /etc/ssh/sshd_config`
* Find the PermitRootLogin line and edit to `no`
* `$ sudo service ssh restart`
#### Configure UFW
* `$ sudo ufw allow 2200/tcp`
* `$ sudo ufw allow 80/tcp`
* `$ sudo ufw allow 123/udp`
* `$ sudo ufw enable`
#### Install Apache and GIT
* `$ sudo apt-get install apache2`
* `$ sudo apt-get install libapache2-mod-wsgi python-dev`
* `$ sudo apt-get install git`
#### Enable mod_wsgi
* `$ sudo a2enmod wsgi`
* `$ sudo service apache2 start`
#### Setup Folders
* `$ cd /var/www`
* `$ sudo mkdir catalog`
* `$ sudo chown -R grader:grader catalog`
* `$ cd catalog`
#### Clone Catalog Project
* `$ git clone https://github.com/Persapiens7/Item-Catalog-master.git catalog`
#### Create .wsgi file
* `$sudo nano catalog.wsgi`
```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'supersecretkey'
```
* Rename the project.py to `__init__.py`
#### Virtual Machine
* `$ sudo pip install virtualenv`
* `$ sudo virtualenv venv`
* `$ source venv/bin/activate`
* `$ sudo chmod -R 777 venv`
#### Install flask and other packages
* `$ sudo apt-get install python-pip`  
* `$ sudo pip install Flask`  
* `$ sudo pip install Requests`  
* `$ sudo pip install httplib2`  
* `$ sudo pip install sqlalchemy`  
* `$ sudo pip install psycopg2`  
* `$ sudo pip install oauth2client`  
* `$ sudo pip install render_template`  
* `$ sudo pip install sqlalchemy_utils`  
* `$ sudo pip install redirect`  
* Use `$ sudo nano __init__.py` to change the client_secrets.json line to `/var/www/catalog/catalog/client_secrets.json`
* Change the host to 18.221.145.45 and port to 80
#### Configure virtual host

* `$ sudo nano /etc/apache2/sites-available/catalog.conf`
```
    <VirtualHost *:80>
    ServerName 34.219.79.240
    ServerAlias ec2-34-219-79-240.us-west-2.compute.amazonaws.com
    ServerAdmin admin@34.219.79.240
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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
```

#### Database
* `$ sudo apt-get install libpq-dev python-dev`  
* `$ sudo apt-get install postgresql postgresql-contrib`  
* `$ sudo su - postgres`  
* `$ psql`

* `$ CREATE USER catalog WITH PASSWORD 'password';`
* `$ ALTER USER catalog CREATEDB;`
* `$ CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to database `$ \c catalog`
* `$ REVOKE ALL ON SCHEMA public FROM public;`
* `$ GRANT ALL ON SCHEMA public TO catalog;`
* Quit the postgres command line: `$ \q` and then `$ exit`

* `$ nano __init__.py`
     Edit `database_setup.py`, and `menus.py` files to change the database engine from `sqlite:///restaurantmenu.db` to `postgresql://catalog:udacity@localhost/catalog`
* Add `ec2-34-219-79-240.us-west-2.compute.amazonaws.com` to Authorized JavaScript Origins and Authorised redirect URIs on Google Developer Console.
* `$ sudo service apache2 restart`

### References:
https://github.com/chuanqin3/udacity-linux-configuration

https://github.com/mulligandev/Udacity-Linux-Configuration
