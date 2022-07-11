# Introduction
This is a guide to setup a new installation of Django, running in an Ubuntu 22.04 Vagrant Box, on an Ubuntu 22.04 Host machine. There are probably better alternatives (the official Django Docker images are good) but for me personally this is the easiest and cleanest way to get up and running locally with Django, and still have traditional command line tools.

## Clone Repository

Clone Github repository to your local development machine:

* SSH: `git clone git@github.com:hamishcomau/vagrant-django.git`

## Project Name

Search for all instances of `yourapp` and replace with your preferred project name.

## Deploy Vagrant Box

This requires both Virtualbox and Vagrant to be installed on your local machine:

* https://www.virtualbox.org/wiki/Downloads
* https://www.vagrantup.com/downloads.html

Once installed, we can simply run the Vagrant up command to setup the virtual machine and run the automated deployment script:

* `vagrant up`

Edit your local hosts file (not the hosts file on the Vagrant virtual machine) and add the following line:

* `sudo nano /etc/hosts`
* `192.168.56.10 yourapp.local`

The deployment process may take a bit of time to download the Vagrant box and step through the shell commands, once complete we can access the virtual machine:

* `vagrant ssh`

## Create Django Project

* `vagrant ssh -c 'cd /yourapp && django-admin startproject yourapp .'`

## Import OS
Add the following to the top of `yourapp/settings.py`:
`import os`

## Database Connection
Replace the `DATABASES = ...` in `yourapp/settings.py` with the following:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'yourapp',
        'USER': 'postgres',
        'PASSWORD': 'admin',
        'HOST': 'localhost',
        'PORT': '',  # Set to empty string for default.
    }
}
```
## Allowed Hosts
Replace the `ALLOWED_HOSTS = ...` in `yourapp/settings.py` with the following:
`ALLOWED_HOSTS = ['*']`

## Installed Apps
Add the following to the `INSTALLED_APPS = ...` in `yourapp/settings.py`:

```
"sslserver",
"yourapp"
```

## Run Migrations
cd /yourapp
python3 manage.py makemigrations yourapp
python3 manage.py migrate

## Create Superuser
python3 manage.py createsuperuser

## Collect Static
TO BE ADDED

## Start Development Server

SSH Into Vagrant Box and Start the Django development server:

* `vagrant ssh -c 'cd /yourapp && python3 manage.py runsslserver 0.0.0.0:8080'`

Load the domain you wish to preview in browser, for example:

* `https://yourapp.local:8080`

Browsers will throw an error message stating the certificate is invalid. This is normal, you can proceed to the test domain and/or add the domains to your exceptions list.

---

## Post Installation
========================

Deploy to production server:

* `git pull && python3 manage.py collectstatic`

Drop database on production server which will have active connections:

* `psql -h localhost postgres postgres`
* `UPDATE pg_database SET datallowconn = 'false' WHERE datname = 'yourapp';`
* `SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'yourapp';`
* `DROP DATABASE yourapp;`
* `\q`

Create New PostgreSQL database:

* `sudo -u postgres createdb yourapp`

Upgrade Packages:

* `sudo pip3 install --upgrade -r requirements.txt`

Import PostgreSQL database dump:

* `sudo -u postgres psql yourapp < yourapp.psql`

Export PostgreSQL database:

* `sudo -u postgres pg_dump yourapp > yourapp.psql`

Delete a PostgreSQL database if required to run migrations or import a database:

* `sudo -u postgres dropdb 'yourapp'`
* `sudo -u postgres createdb yourapp`
* `sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'admin';"`

View PostgreSQL databases to ensure encoding is set to UTF8:

* `sudo -u postgres psql postgres`
* `\l`

---
