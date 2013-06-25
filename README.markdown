# Loogica Simple Environment Setup Tools

## Motivation

If you just want to easily setup a linux environment ready to run
Python wsgi compatible apps, this kit can help you.

**If you're not from Brasil, maybe you'll have to change `server/server_setup.sh`
  timezone, currently configured to America/Sao_Paulo**

## Django Apps

This kit is intended to work with Python WSGI Apps but currently we only use and test it against
Django apps.

### Improved Bootstraping
This is how we bootstrap our Django Projects

```sh
chmod +x boom
./boom project_name app_name /tmp
```

## How it works?

There are 3 phases:

1. Environment Settings.
2. Setup Remote Access.
3. Configure a Python Wsgi Friendly machine.
4. Install envkit in your project.
5. Setup a machine to run some specific app.

### Environment Settings:

Loogica Envkit will look for environment files over `env/`. If it finds a file
named `stage.cfg` it'll configure and environment called `stage` that will work
as a regular fabric environment.

This is the template for a environment file:

```ini
[environment]
address=
admin_user=root
project_user=

[application]
name=
```

#### Admin Vs. Project User

An Admin can do basically everything since it's a sudoer.

A Project User can only execute tasks related to a project like:

* Deploy and Restart.
* Read Logs.
* Database tasks per project.

### Remote Access

Given a Root password, you already can setup what we call Admin and Project acctouns on your machine.

To create an remote admin account given a public key:

```sh
fab environment config_user:[pub_key_path]
```

If you'll use your own public key just:

```sh
fab environment config_user
```

#### Remove Remote account

```sh
fab environment remove_user:username
```

### Preparing


### Creating 

This task will ask for a IP Address and at the end, the username picked in the previous
step will be able to connect with this machine using your public key. You'll also have
to choose between install or not Postgresql

```sh
fab config server_bootstrap:loogica,loogica.net,felipecruz@loogica.net
```

## Installing envkit on your projet

```sh
./install_kit.sh SERVERNAME_OR_IP APP_NAME /full/path/to/app/root
```

## Running per-project Tasks

After the previous phase, you should have `env/deploy.cfg` in your project root. Change
it if needed but the default file works fine for initial setups.

### Go to your project root

```sh
cd /full/path/to/app/root
```

### Setup App environment

```sh
fab prod setup_app
```

### Create App database

**dbname should be same as project name**

```sh
fab prod postgres_db_create:dbuser,dbname,password
```

### Setup Nginx Config

```sh
fab prod nginx_setup
```

## Deploy restrictions

Your project root must have a Makefile in order to make the deploy possible. Here's an example:

```Makefile
server_dbinitial:
	python manage.py syncdb --noinput && python manage.py createsuperuser --user admin --email admin@admin.com

migrate_no_input:
	python manage.py migrate --noinput $(APPS)

update_deps:
	sudo pip install -r requirements.txt
```

If you're running, for instance, a Flask app, some targets may be empty.

### First Deploy

```sh
fab prod first_deploy:HEAD
```

### Regular deploy

```sh
fab prod deploy:HEAD
```

## Django Make Targets

Create a new app based on our app template.

```sh
make new_app app_name
```

Add this app to settings.py `INSTALLED_APPS` and create
initial migration for it

```sh
make app_schemamigration app_name
```

Finally, update you `Makefile` and modify it's first line adding your app
separeted by space like:

```Makefile
APPS=app1 app2
```

## License

```
This is free and unencumbered software released into the public domain.

Anyone is free to copy, modify, publish, use, compile, sell, or
distribute this software, either in source code form or as a compiled
binary, for any purpose, commercial or non-commercial, and by any
means.

In jurisdictions that recognize copyright laws, the author or authors
of this software dedicate any and all copyright interest in the
software to the public domain. We make this dedication for the benefit
of the public at large and to the detriment of our heirs and
successors. We intend this dedication to be an overt act of
relinquishment in perpetuity of all present and future rights to this
software under copyright law.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

For more information, please refer to <http://unlicense.org/>
```
