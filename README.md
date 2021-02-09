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
```




