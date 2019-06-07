# Amazon Lightsail Ubuntu Web Server Setup

Details for setting up the Catalog Flask App on a newly created Amazon Lightsail Ubuntu VM

## Server Security

1) Update packages

    sudo apt-get update
    sudo apt-get upgrade

2) Check status of `ufw` to make sure messing up the ports won't brick the server

    sudo ufw status

3) Edit Amazon firewall settings to add new ports SSH TCP 2200 and NTP UDP 123 from the below URL
https://lightsail.aws.amazon.com/ls/webapp/us-east-1/instances/asaeles-web-server/networking
4) Edit SSH config to make sure password login is disabled and to change port to 2200

    sudo nano /etc/ssh/sshd_config

5) Restart SSH service

    sudo service ssh restart

6) Test SSH on the new port and make sure it is working

7) Setup `ufw`

    sudo ufw default deny incoming
    sudo ufw defualt allow outgoing
    sudo ufw allow ssh
    sudo ufw allow www
    sudo ufw allow ntp/tcp
    sudo ufw allow 2200/tcp

8) Enable `ufw`

    sudo ufw enable
    sudo ufw status

## Create User

9) Add new user `grader`

    sudo adduser grader

10) Create new key pair using PuTTY Key Gen for Windows then copy the public key and save the private key to "grader.ppk"
11) Add public key to authorized keys copy/paste

    su grader
    cd ~
    mkdir .ssh
    nano .ssh/authorized_keys
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys

12) Add new user to `sudoers` by copying any existing file

    sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
    sudo nano /etc/sudoers.d/grader

13) Test login and `sudo` using user `grader`

## Web Server Setup

14) Install Apache and PostgresSQL

    sudo apt-get install apache2
    sudo apt-get install libapache2-mod-wsgi
    sudo apt-get install postgresql
    sudo apt-get install libpq-dev
    sudo apt install redis-server
    nohup redis-server &

15) Install Flask and SQL Alchemy

    sudo apt-get install python-pip
    pip install --upgrade pip
    sudo su
    pip install flask
    pip install flask_httpauth
    pip install requests
    pip install sqlalchemy
    pip install psycopg2
    pip install redis
    exit

16) Configure catalog app using `catalog.conf` and `catalog.wsgi`

    sudo rm /etc/apache2/sites-enabled/000-default.conf
    sudo nano /etc/apache2/sites-available/catalog.conf
    sudo ln -s /etc/apache2/sites-available/catalog.conf /etc/apache2/sites-enabled/catalog.conf
    sudo mkdir /var/www/catalog/
    sudo nano /var/www/catalog/catalog.wsgi
    sudo apache2ctl restart

## Setup Catalog App

17) Setup DB

    sudo -u postgres psql

Then run the following PSQL commands replacing 'password'

    CREATE USER catalog NOCREATEDB NOCREATEUSER PASSWORD 'password';
    CREATE DATABASE catalog OWNER catalog;

18) Setup catalog app as per the readme file for repository
    cd /var/www/catalog
    sudo git clone https://github.com/asaeles/catalog.git
    cd catalog
    sudo cp catalog_default.ini catalog.ini
    sudo nano catalog.ini
    sudo nano templates/index.html
