# django

For now I have kepy:

```
(env) ubuntu@ip-172-31-5-30:~/django$ egrep '^DEBUG|^ALLOW' mysite/settings.py
DEBUG = True
ALLOWED_HOSTS = ["*"]
(env) ubuntu@ip-172-31-5-30:~/django$
```

As you can verify how the site works in all ways, later in production this has to be single IP allowed, and DEBUG is False.


## This site is running on AWS EC2 instance (free tier)

Load the below URL:

#### http://ec2-13-126-207-57.ap-south-1.compute.amazonaws.com:8000/polls/
=======================================================================

## It uses below addons to render as webserver and startups

##### NGINX -> serves as weserver engine

##### GUNICORN -> used to interact with Django application in helping starting and stopping the application server

##### SUPERVISOR -> Supervisor package 'supervisorctl' is used to start and stop GUNICORN


It internally uses below command:

gunicorn --bind 0.0.0.0:8000 mysite.wsgi:application

Static files are also rendered by nginx


## Below are the configurations:

```
(env) ubuntu@ip-172-31-5-30:~/django$ cat /etc/supervisor/conf.d/gunicorn.conf
[program:gunicorn]
directory=/home/ubuntu/django
command=/home/ubuntu/env/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/django/app.sock mysite.wsgi:application
autostart=true
autorestart=true
stderr_logfile=/var/log/gunicorn/gunicorn.err.log <==== Logs are seperately managed
stdout_logfile=/var/log/gunicorn/gunicorn.out.logA <=== Standard output also loged seperately

[group:guni]
programs:gunicorn
(env) ubuntu@ip-172-31-5-30:~/django$

```

Process running OK

```
(env) ubuntu@ip-172-31-5-30:~/django$ sudo supervisorctl status
guni:gunicorn                    RUNNING   pid 1713, uptime 0:29:37 <==== gunicorn runnins
(env) ubuntu@ip-172-31-5-30:~/django$ 
```




## nginx config:

```
(env) ubuntu@ip-172-31-5-30:~/django$ cat /etc/nginx/sites-available/django.conf
server {
    listen 8000;
    server_name ec2-13-126-207-57.ap-south-1.compute.amazonaws.com;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/django/app.sock; <========= Binds socket
    }
    location /static/ { <============================================== Helps to feed static files
        autoindex on;
        alias /home/ubuntu/django/polls/static/;
    }
}
(env) ubuntu@ip-172-31-5-30:~/django$ 

```


