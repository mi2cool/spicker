# Deploying multiple apps on single server with nginx and gunicorn

This readme is based on the following tutorial:
[Tutorial](https://www.youtube.com/watch?v=koo3bF2EPqk "This is a tooltip")

### Setup python environment
Install and setup the python environement. 

    $ python -m pip install virtualenv
    $ python -m virtualenv appname-env
    $ source apnname-env/bin/activate
    $ python -m pip install -r app-project-dir/requirements.txt
    $ python -m pip install gunicorn


## Create services for application [#setup_services]
### Create and configure socket

    $ sudo nano /etc/systemd/system/appname.socket

Copy following configuration to _appname.socket_. 

    [Unit]
    Description=gunicorn socket
    
    [Socket]
    ListStream=/run/appname.sock
    
    [Install]
    WantedBy=sockets.target 


Create symbolic link to activate configuration:
    $ sudo ln -s /etc/nginx/sites-available/pacman /etc/nginx/sites-enabled


### Create and configure service
Create service file.
    $ sudo nano /etc/systemd/system/appname.service

Copy configuration to _appname.service_.

    [Unit]
    Description=gunicorn daemon
    Requires=appname.socket
    After=network.target
    
    [Service]
    User=root
    Group=www-data
    WorkingDirectory=/root/webapps/app_project_dir
    ExecStart=/root/webapps/app_project_dir/app-env/bin/gunicorn --workers 3 --bind unix:/run/appname.sock app_project_name.wsgi:application
    
    [Install]
    WantedBy=mutli-user.target

## Install and configure nginx
Create nginx configuration file.

    $ sudo nano /etc/nginx/sites-available/appname
Add configuration to _appname_.

    server{
        listen 80;
        server_name domainname.com;
        location = /favicon.ico {access_log off; log_not_found off;}
        
        client_max_body_size 1000M ;
        
        location / {
            include proxy_params;
            proxy_pass http://unix:/run/appname.sock;
        }
    }
	

Create symbolic link 
```
$ sudo ln -s ../sites-available/muffels_pictures .
$ sudo systemctl restart nginx
```



# Start services
Start services 
    
    $ sudo systemctl start nginx
    $ sudo systemctl enable nginx # enable: service will be started after reboot
    $ sudo systemctl start appname.socket
    $ sudo systemctl enable appname.socket # enable: service will be started after reboot



