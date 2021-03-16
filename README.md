# Oasis 2.0 [![Build Status](https://travis-ci.com/stampylongr/oasisclosed.svg?branch=master)](https://travis-ci.com/stampylongr/oasisclosed)
A modified version of Oasis. Also join our Website! https://le-kokomo.party

# Requirements
```console
$ pip3 install Django urllib3 lxml passlib bcrypt pillow django-markdown-deux django-markdown2
```
  * Latest version of Python
  * Django
  * urllib3
  * lxml
  * passlib
  * bcrypt
  * mysqlclient if you're using mysql
  * pillow
  * markdown

[![forthebadge](https://forthebadge.com/images/badges/made-with-python.svg)](https://forthebadge.com)

[![forthebadge](https://forthebadge.com/images/badges/you-didnt-ask-for-this.svg)](https://forthebadge.com)

# **Usage**
This guide assumes that you are using Ubuntu 18.04 or Ubuntu 20.04. If you are not using Ubuntu 18.04/20.04, look up a guide for your platform that uses Gunicorn, PostgreSQL, NGINX, and Django.

By the way do not forget to create an user on your server. If you do not know how to do it, check [this](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu-18-04-quickstart) guide by DigitalOcean.

To install the required packages, use the following commands:
```console
$ sudo apt update
$ sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl
```
To create the database needed for Oasis, run the following command:
```console
$ sudo -u postgres psql
```
Inside the PostgreSQL prompt, do the following (replacing 'password' with your password):
```sql
CREATE DATABASE oasis_db;
CREATE USER oasis WITH PASSWORD 'password';
ALTER ROLE oasis SET client_encoding TO 'utf8';
ALTER ROLE oasis SET default_transaction_isolation TO 'read committed';
ALTER ROLE oasis SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE oasis_db TO oasis;
\q
```
Now run the following to install virtualenv (so we can find the executables easily):
```console
$ sudo -H pip3 install --upgrade pip
$ sudo -H pip3 install virtualenv
```
Now clone this repo with:
```console
$ git clone https://github.com/SMF9/oasis
$ cd oasis
```
And create the virtualenv with:
```console
$ virtualenv oasis_env
```
Now run the following to activate the environment:
```console
$ source oasis_env/bin/activate
```
And install the requirements with the following:
```console
$ pip install psycopg2
$ pip install gunicorn
$ pip install requests
```
Edit your settings using:
```console
$ nano ~/oasis/closedverse/settings.py
```
Remember to change the user and password for the user in the settings!
For the Django Secret Key (KEEP SECRET!!!!!), you can generate one here. https://djecrety.ir/
Also, don't forget to add your domain to the ALLOWED_HOSTS portion of the settings.py

Now run the following to make the database:
```console
$ python manage.py makemigrations
$ python manage.py makemigrations closedverse_main
$ python manage.py migrate
```

And run the following to make the admin user  (if you want an admin user this step is necessary):
```console
$ python manage.py createsuperuser
```

Now run the following to collect the static assets:
```console
$ python manage.py collectstatic
```

Now type the following to get out of the virtualenv:
```console
$ deactivate
```

## Systemd files
Now we are going to make the systemd files to easily control the server.

Run the following to start editing `gunicorn.socket`:
```console
$ sudo nano /etc/systemd/system/gunicorn.socket
```
Now paste the following inside the file:
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

Now use CTRL+X and type `y` and then ENTER to save the file.

Run the following to create the service:
```console
$ sudo nano /etc/systemd/system/gunicorn.service
```

Paste the following inside the file:
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=YOUR_USERNAME_HERE
Group=www-data
WorkingDirectory=/home/YOUR_USERNAME_HERE/oasis
ExecStart=/home/YOUR_USERNAME_HERE/oasis/oasis_env/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          closedverse.wsgi:application

[Install]
WantedBy=multi-user.target
```
Remember to replace YOUR_USERNAME_HERE with your own username.

Press CTRL+X then `y` and then ENTER to save the file.

Now run the following to start and enable the service on boot:
```console
$ sudo systemctl start gunicorn.socket
$ sudo systemctl enable gunicorn.socket
```

To test the socket, run the following:
```console
$ curl --unix-socket /run/gunicorn.sock localhost
```

If it outputs something that starts with `<!doctype html>`, then everything is fine.

Run the following to start the process:
```console
$ sudo systemctl daemon-reload
$ sudo systemctl restart gunicorn
```

Now, for NGINX.

Open a file using this command below:
```console
$ sudo nano /etc/nginx/sites-available/oasis
```

Paste the following inside:
```nginx
server {
    listen 80;
    server_name insert_server_domain_here;

   location = /favicon.png { access_log off; log_not_found off; }
    location /s/ {
        alias /home/YOUR_USERNAME_HERE/oasis/s/;
    }
    location /i/ {
        alias /home/YOUR_USERNAME_HERE/oasis/i/;
    }

    
    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

Now link the files:
```console
$ sudo ln -s /etc/nginx/sites-available/oasis /etc/nginx/sites-enabled
```

And run the NGINX test:
```console
$ sudo nginx -t
```

Now you need to restart NGINX, and enable the firewall:
```console
$ sudo systemctl restart nginx
$ sudo ufw allow 'Nginx Full'
```

That's it!

If you want SSL, use [this](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04) guide by DigitalOcean.

# Copyright
Copyright 2021 SMF9, all rights reserved to their respective owners. (Arian Kordi, Pip, PF2M, Nintendo, Hatena Co Ltd.)

