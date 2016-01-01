#Flask App Deployment

###AWS Instance
1. First create an AWS instance.
2. save the pem key to /home/your_username/.ssh/
3. edit the inbound security rules under Security Groups dropdown to include HTTP with port range 80. This is so you can access the instance from the net.
4. change the security of the key using: ```chmod 400 name_of_key_pair.pem```

###Getting inside the terminal
for ubuntu virtual instance:
* To get inside the instance, you can use the public ip: ```ssh -i name_of_key_pair.pem ubuntu@public_ip```
* or you can use the public dns: ```ssh -i name_of_key_pair.pem ubuntu@public_dns```

To transfer files inside, use: ```scp -i name_of_key_pair.pem samplefile.txt ubuntu@public_ip:<path within instance>```

To transfer folders, replace **samplefile.txt** with ```-r path/to/folder``` -r for recursive transfer.

###Updating the system instance
Inside the instance, make sure to run commands below so that everything gets updated before we begin our work:
* ```sudo apt-get update && sudo apt-get upgrade```

###Install/Setup PostgreSQL
link guide: [https://help.ubuntu.com/community/PostgreSQL](https://help.ubuntu.com/community/PostgreSQL)

Install postgres: ``` sudo apt-get install postgresql postgresql-contrib```

To start off, we need to change the PostgreSQL postgres user password; we will not be able to access the server otherwise. As the “postgres” Linux user, we will execute the psql command.

1. On the terminal, type ```sudo -u postgres psql postgres```
2. Set a password for the "postgres" database role using the command: ```\password postgres```
3. To create your db with "postgres" as the user: ```sudo -u postgres createdb name_of_db```

The default when calling ```psql``` is the current user as the postgres user accessing a db that has the same name as the user. With that to make things easier:

1. create a superuser that has the same name as your current ubuntu user: ```sudo -u postgres createuser --superuser $USER```
2. access psql: ```sudo -u postgres psql```
3. inside psql, to change password of current user: ```\password <the name of the the user>```
4. to quit psql ```\q```
5. outside psql, to create the db: ``` sudo -u $USER createdb $USER```

Connecting to your own database should be as easy as ```psql``` now.
* creating other db will now be like: ``` createdb amarokdb;```

To access gain access on a local database without it asking for username and password:

1. access the **pg_hba.conf** file:  ```sudo nano /etc/postgresql/9.3/main/pg_hba.conf``
2. change the line:
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer
```
to
```
# Database administrative login by Unix domain socket
local   all             postgres                                trust
```
&nbsp;&nbsp;3. &nbsp;reload postgreSQL server configuration: ```sudo /etc/init.d/postgresql reload```

###Install pip/virtualenv/virtualenvwrapper
Install pip (for easy module installation): ```sudo apt-get install python-pip```

Install virtualenv (for easy dependency management): ```sudo pip install virtualenv```

Install virtualenvwrapper (for easier dependency management): ```sudo pip install virtualenvwrapper```

To make virtualenvwrapper work, place the following lines at ~/.bashrc:
```
export PATH=/usr/local/bin:$PATH
source /usr/local/bin/virtualenvwrapper.sh
```
Then reload ~/.bashrc: ```. ~/.bashrc```

Install python-dev (needed by psycopg2): ```sudo apt-get install python-dev```

Install python-dev (to make **psycopg2** work which is then used by **sqlalchemy**): ```sudo apt-get install libpq-dev``` and ```sudo pip install psycopg2``` 

###Transferring your local app to AWS Instance
be careful to change all the paths affected by the change in directory.

```scp -i name_of_key_pair.pem -r path/of/app/folder ubuntu@public_ip:<path within instance>```

create a virtualenv for all the dependencies: ```mkvirtualenv <my_virtualenv> && workon <my_virtualenv>```

populate virtualenv with dependencies: ```pip install -r requirements.txt```

Check if everything is running by running the tests again (if ever you created a few).

###Running nginx
Installing nginx: ```sudo apt-get install nginx```

Delete the default config and create a new config for the app that will listen to port 80 (the HTTP port that we set while creating the AWS instance) and will then pass it to port 8000 (the default port of gunicorn):
```
sudo rm /etc/nginx/sites-enabled/default
sudo nano /etc/nginx/sites-available/flask_project
```
Inside the flask project, write:
```
  server {
    listen 80;
    server_name example.org;
    access_log  /var/log/nginx/example.log;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  }
```
Create a symbolic link to sites enabled: ```sudo ln -s /etc/nginx/sites-available/flask_project /etc/nginx/sites-enabled/flask_project```

Restart nginx to start the new changes with either method:
* ```sudo /etc/init.d/nginx restart```
* ```sudo service nginx restart```

###Running gunicorn
In production, we won't be single threaded development servers. We will instead use a dedicated application server called **gunicorn**.

For Flask: ```gunicorn run:app``` the word run being the name of wsgi script (run.py).

A better way to run gunicorn is from bash script. That way it can easily be run by Supervsior:


create a text file: ```nano gunicorn_start```
```
#!/bin/bash

NAME="gunicorn_nae"                                                                     # name of the application
FLASKDIR="path/to/app                                                                   # flask project directory
USER=antonio                                                                            # the user to run as
GROUP=antonio                                                                           # the group to run as
NUM_WORKERS=3                                                                           # how many worker processes should Gunicorn spawn
FLASK_WSGI_MODULE=run                                                                   # WSGI module name

echo "Starting $NAME as `whoami`"

# Activate the virtual environment
# Do not use virtualenvwrapper inside here. It will keep on looking for\
# the virtualenv folder at main root folder of linux and not inside /home.
source /home/antonio/.virtualenvs/nodes_and_edges_5/bin/activate

# Start the Green Unicorn
# Programs meant to be run under supervisor should not daemonize themselves (do not use --daemon)
cd $FLASKDIR

gunicorn ${FLASK_WSGI_MODULE}:app \
    --name $NAME \
    --workers $NUM_WORKERS \
    --log-level=debug \
    --log-file=-
```
test if its working by running: ```./gunicorn_start```

###Using Supervisor to daemonize tasks
We will be using Supervisor to automate processes and running on the backgroun.

To install Supervisor: ```sudo apt-get install supervisor```

to create a configuration file: ``` sudo nano /etc/supervisor/conf.d/<name_of_project>.conf```

Include the commands that are supposed to be ran inside the conf file:
```
[program:gunicorn]
command = /path/to/app/scripts/gunicorn_start                                         ; Command to start app.
stdout_logfile = /path/to/app/logs/gunicorn_supervisor.log                            ; Where to write log. messages
stderr_redirect = true                                                                ; To also log the errors
user = ubuntu                                                                         ; User to run as.
```

To check if the config have changed: ```sudo supervisorctl reread```
to run the update of the project (and then running them): ```sudo suprvisorctl update```

You can also check the status of your app or start, stop and restart using supervisor:
```
$ sudo supervisorctl status hello                       
hello                            RUNNING    pid 18020, uptime 0:00:50
$ sudo supervisorctl stop hello  
hello: stopped
$ sudo supervisorctl start hello                        
hello: started
$ sudo supervisorctl restart hello 
hello: stopped
hello: started
```

#TO BE ADDED SOON..
* ~~create a bash script for gunicorn~~
* ~~use supervisor~~
* ~~using supervisor with virtualenvs~~
