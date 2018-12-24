# linux-server-configuration 

## Description
This up to date guide is designed to help other Udacity students with the final project for the FSND. This will give a step by step description into deploying a Flask application using Ubuntu and Apache with Amazon Lightsail.

## IP
- IP Address: http://35.159.31.182.xip.io/

## Amazon Lightsail Set Up
1. First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
2. Create an instance.
Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
3. Choose an instance image: Ubuntu
Lightsail supports a lot of different instance types. An instance image is a particular software setup, including an operating system and optionally built-in applications.

4. Choose your instance plan.
The instance plan controls how powerful of a server you get. It also controls how much money they want to charge you. For this project, the lowest tier of instance is just fine
5. Give your instance a hostname.
Every instance needs a unique hostname. You can use any name you like, as long as it doesn't have spaces or unusual characters in it. Your instance's name will be visible to you and to the project reviewer.
7. It's running; let's use it!
Once your instance has started up, you can log into it with SSH from your browser.

The public IP address of the instance  `35.159.31.182`.


# Linux Configuration
 1. Download your instance Private Key from your profile.
 2. in your local machine, cd to `‎⁨Macintosh HD⁩ ▸ ⁨Users⁩ ▸ [your user] ▸ .ssh`.
 3. move your private key file into this directory with the name `udacity.pem`.
 4. name it 'udacity.rsa'
 5. change permission using `chmod 600 ~/.ssh/udacity.pem`.
 6. log into our Amazon Lightsail Server: `$ ssh -i ~/.ssh/udacity.pem ubuntu@[PUBLIC IP ADDRESS]`.
 7. log in as root user: `sudo su -`.



## create grader account
 1. `sudo adduser grader`.
 2. `sudo nano /etc/sudoers.d/grader`
  - add the following line `ALL=(ALL:ALL) ALL` to add grader as a user.

## update virtual machine packages
1. `sudo apt-get update`.
2. `sudo apt-get upgrade`.
3. ` sudo apt-get install finger`.

## generate public key
1. in another terminal write this `ssh-keygen -f ~/.ssh/udacity.rsa` it will ask for password that grader will use to access the machine.
2. `cat ~/.ssh/udacity_key.rsa.pub` to find the public key of that password. COPY IT.
3. Back to server terminal, ` cd /home/grader` then create a ssh file for saving public keys with
  - `mkdir .ssh`
  - `touch .ssh/authorized_keys`
  - `nano .ssh/authorized_keys` :paste the public key.
  - change file permissions. `sudo chmod 700 /home/grader/.ssh and $ sudo chmod 644 /home/grader/.ssh/authorized_keys`
  - change file owner `sudo chown -R grader:grader /home/grader/.ssh`
  - restart server configuration `sudo service ssh restart`.
  - disconnect.

## log in as grader
using `ssh -i ~/.ssh/udacity.rsa grader@35.159.31.182`
- enable key authentication instead of password authentication and resetting PasswordAuthentication to NO and change port to 22200 using `sudo nano /etc/ssh/sshd_config`.
- `sudo service ssh restart` to refresh server with new configuration
- BEFORE disconnecting and accessing server with port 2200, we need to setup firewall or else we will lose our access to the server:
  - `sudo ufw allow 2200/tcp`
  - ` sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`
- Diconnect and log in using `ssh -i ~/.ssh/udacity.rsa grader@35.159.31.182 -p 2200`
# Application Deployment
 1. we need ti install packages required to setup our application :
  - `sudo apt-get install apache2`
  - `sudo apt-get install libapache2-mod-wsgi python-dev`
  - `sudo apt-get install git`
2. Enable mod_wsgi using$ `sudo a2enmod wsgi` then `sudo service apache2 restart`.
3. Set up the folder structure :
  - `cd /var/www`
  - `sudo mkdir catalog`
  - ` sudo chown -R grader:grader catalog`
  - `cd catalog`
  - ` git clone [Github Link] catalog`
4. create .wsgi file `sudo nano catalog.wsgi` in the same directory.
  - add the follwoing inside it
  ```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'supersecretkey'
 ```
    
  
5. Rename the `main.py` to `__init__.py`
6. installing the virtual machine:
  - ` sudo pip install virtualenv`
  - `sudo virtualenv venv`
  - `source venv/bin/activate`
  - `sudo chmod -R 777 venv`
7. install Flask and other packages:
  - ` pip install psycopg2-binary`
  - `pip install psycopg2t`
  - `pip install Flask-SQLAlchemy`
  - `pip install requests`
  - `pip install oauth2client`
8. nano to  `__init__.py` and change `client_secrets.json` path to `/var/www/catalog/catalog/client_secrets.json`
9. setup server configuration with `sudo nano /etc/apache2/sites-available/catalog.conf`
10. write the following inside if it:
```
<VirtualHost *:80>
    ServerName [YOUR PUBLIC IP ADDRESS]
    ServerAlias [YOUR AMAZON LIGHTSAIL HOST NAME]
    ServerAdmin admin@[YOUR PUBLIC IP ADDRESS]
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

## setup the database
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`

  creating the databse :
  - `CREATE USER catalog WITH PASSWORD [your password];`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - Connect to database  `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - ` \c and then exit`

  - Now go and change database_setup and seeder and __init__ files to have instead of `sqlite:///databasename.db` to `'postgresql://catalog:catalog@localhost/catalog` with username `catalog` and password `catalog`.


## setup Google oAuth properly :
  - go to google console and add authorized domains `xip.io`.
  - Add authorized Javascript origins: `http://35.159.31.182.xip.io`
  - Add authorised redirect URIs:             `http://54.93.239.136.xip.io/gconnect` and `http://54.93.239.136.xip.io/login`
  - download the updated JSON file, copy its content and paste it in server copy : `sudo nano client_secrets.json`. make sure clients_secrets provided in the JSON otherwise it won't work!.


### FINALYY :
`sudo service apache2 restart` and open up your ip address in the browser with extension `xip.io`.


### References:

 - https://github.com/callforsky/udacity-linux-configuration
 - https://github.com/rrjoson/udacity-linux-server-configuration/blob/master/README.md
