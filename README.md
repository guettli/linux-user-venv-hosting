# linux-user-venv-hosting

My simple way to host Python applications in a virtual-environment in a linux-user.

I have a simple VPS (Virtual-Private Server) and use it to run serveral applications.

I know that the future will be container based, but up to now I am happy with this simple
way of hosting simple Python/Django based applications.

```
root@server# adduser foo
... (create a long random password. Which I don't write down, since I never use password-auth)

root@server# su - foo
foo@server> mkdir .ssh
foo@server> vi .ssh/authorized_keys
  add my .ssh/id_rsa.pub content
```

```
# check passwordless ssh access:
me@laptop> ssh foo@server 

# Create virtualenv in $HOME
foo@server> python3 -m venv .

foo@server> vi .env
add:
export PROJECT=${USER}
export PGDATABASE=${USER}
export PGUSER=${USER}
export DEBUG=False
export SECRET_KEY='****-use-your-own-****'
export DJANGO_SETTINGS_MODULE=mysite.settings

foo@server> vi .bashrc
add:
export EDITOR=vim
. bin/activate
. .env

foo@server> logout
```

```
# connect again
ssh foo@server

# Check that the environment variables are set:
foo@server> echo $PGDATABASE

# Update basics
foo@server> pip install -U pip wheel

# Install my code
foo@server> pip install -e git+ssh://git@github.com/YOU/foo.git#egg=foo
```

```
# DB
hroot@server# su - postgres

# If the linux user name is equal to the database user name, then
# you don't need to configure a password, if you use PostgreSQL
# via localhost.

postgres@server> createuser foo
postgres@server> createdb -O foo foo
```

```
# If you updated your code, you can get the new code like this:
foo@server> cd ~/src/foo
foo@server> git pull
```

```
# Run Migrations
foo@server> manage.py migrate
```

```
# Systemd: Start gunicorn

# file /etc/systemd/system/gunicorn-foo.service

[Unit]
Description=gunicorn daemon foo
Requires=gunicorn-foo.socket
After=network.target

[Service]
Type=notify
# the specific user that our service will run as
User=foo
Group=foo
# another option for an even more restricted service is
# DynamicUser=yes
# see http://0pointer.net/blog/dynamic-users-with-systemd.html
RuntimeDirectory=gunicorn
WorkingDirectory=/home/foo
ExecStart=/home/foo/bin/gunicorn mysite.wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=5
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

```
# /etc/systemd/system/gunicorn-foo.socket

[Unit]
Description=gunicorn foo socket

[Socket]
ListenStream=/run/gunicorn-foo.sock
# Our service won't need permissions for the socket, since it
# inherits the file descriptor by socket activation
# only the nginx daemon will need access to the socket
User=www-data
# Optionally restrict the socket permissions even more.
# Mode=600

[Install]
WantedBy=sockets.target
```

```
root@server# systemctl enable --now gunicorn-foo.socket
```

Webserver config see: https://docs.gunicorn.org/en/stable/deploy.html



