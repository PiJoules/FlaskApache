# Setting up a Flask Application on Apache
These are instructions on how to setup a flask application on an existing apache server. For an existing apache server, you will just need to proxy HTTP traffic through apache to your flask app. This way, apache can handle the static files (which it's very good at - much better than the debug server built into Flask) and act as a reverse proxy for your dynamic content, passing those requests to Flask.

Also included are instructions on how to run multiple applications on the same server. With this, you can support subdomains on your website, and be able to get the most of your VPN without having to use one VPN for each application you would like to deploy.

## Dependencies
These instructions asume you are on a `Linux` OS using a `Bash` shell. These instructions also assume you have `python`, `git`, `vim` (or some other text editor), and `apache2` installed.

Other dependecies will be installed while running these instructions. I do not assume you have these installed beforehand because these new ones are not required for running the default apache server. This repo also contains a sample Flask application that will be used in this example.

## Instructions
1) You will need to make sure mod_wsgi is installed and enabled. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. On linux, this will install and enable mod_wsgi.
```sh
$ sudo apt-get install libapache2-mod-wsgi # install
$ sudo a2enmod wsgi # enable
```

2) On your terminal, move to the directory `/var/www/`. This directory should exist if you have Apache installed. The existing `/var/www/html` directory can be ignored.
```sh
$ cd /var/www
```

3) Create a directory named after your app (or any name really, though the name of your app may make the most sense) and `cd` into it. For this example, the name `FlaskApache` will be used.
```sh
$ mkdir FlaskApache
$ cd FlaskApache
```

4) Clone the git repo of this tutorial in the `FlaskApache` directory.
```sh
$ git clone https://github.com/PiJoules/FlaskApache.git
```

This will now create a `FlaskApache` directory in your existing `FlaskApache` directory. From this point on, the first `FlaskApache` directory will be referred as the `parent` one and the cloned one as the `nested` one. Your file structure should look like this now.
```sh
FlaskApache
└── FlaskApache
    ├── __init__.py
    ├── lib
    │   └── README.md
    ├── private
    │   ├── __init__.py
    │   └── README.md
    ├── README.md
    ├── requirements.txt
    ├── setup.sh
    ├── static
    │   ├── assets
    │   ├── css
    │   └── js
    ├── templates
    │   └── index.html
    └── vendor.py
```

5) In the parent directory, create a file called `FlaskApache.wsgi` and write the following content to it
```sh
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApache/")
# Here `FlaskApache` is the parent one, not the nested one
from FlaskApache import app as application
```

Now your file structure should look like this
```sh
FlaskApache
├── FlaskApache
│   ├── __init__.py
│   ├── lib
│   │   └── README.md
│   ├── private
│   │   ├── __init__.py
│   │   └── README.md
│   ├── README.md
│   ├── requirements.txt
│   ├── setup.sh
│   ├── static
│   │   ├── assets
│   │   ├── css
│   │   └── js
│   ├── templates
│   │   └── index.html
│   └── vendor.py
└── FlaskApache.wsgi
```

6) Create a file in `/etc/apache2/sites-available` called `FlaskApache`. If runnning on Ubuntu (13.10+), the file will end in `.conf` and be `FlaskApache.conf`.
```sh
$ sudo /etc/apache2/sites-available/FlaskApache
or
$ sudo /etc/apache2/sites-available/FlaskApache.conf
```

And place the following content inside this file
```sh
<VirtualHost *:80>
        ServerName [IP Address or domain name of this server]
        WSGIScriptAlias / /var/www/FlaskApache/FlaskApache.wsgi
        <Directory /var/www/FlaskApache/FlaskApache/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/FlaskApache/FlaskApache/static
        <Directory /var/www/FlaskApache/FlaskApache/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

This sets up the virtualhost for the flask app. Be sure to replace the ServerName with the atcual IP or domain name of the server. If you are running on a local server, you can use `127.0.0.1` as the ServerName. Each of the directories shown above involving the `FlaskApache` directories and files is simply a mapping to each of the locations of each respective file.

7) Enable the virtualhost and restart apache
```sh
$ sudo a2ensite FlaskApache # enable
$ sudo service apache2 restart # restart
```

You may see the following warning
```sh
Could not reliably determine the VPS's fully qualified domain name, using 127.0.0.1 for ServerName
```

You do not need to worry about this, and you will be able to access your virtual host without any further issues.

8) Install the dependencies for this particular flask app using pip. (At this point, I am just setting up stuff for this particular Flask app and am done setting up the apache server. If this is not how you need to setup your app, then this part can be skipped. Just make sure your app has all its dependencies installed and is working. You will not be able to access your app via apache if the app itself doesn't work.)
```sh
$ pip install -r requirements.txt -t lib/ # do this inside the nested FlaskApache directory
```

At this point, you can go to the IP/URL specified in the `ServerName` property and you see your flask application deployed.


## Running Multiple Applications on the same Apache Server
To run another Flask application on the same server, you just need to repeat the above steps but replacing every instance of the path to your first app with your path to the second app. For example, you can duplicate the directory containing the parent `FlaskApache` in the same directory it's in, but rename it `FlaskApache2`. You would then adjust the path inside the `FlaskApache2/FlaskApache.wsgi` file by replacing it with this file path, and add another `.conf` file in the `/etc/apache2/sites-available/` directory similar to the first one, but adjusting the paths appropriately and replacing the server name whith whatever the name of the domain of your application is. If you would like to have a subdomain `eggs.FlashApache.com`, you would place `eggs.FlaskApache.com` in the server name.

Lastly, to run the application, you need to enable it and restart the server:
```sh
$ sudo a2ensite FlaskApache # enable
$ sudo service apache2 restart # restart
```


## Closing Stuff
- For your application, just be sure to replace all instances of `FlaskApache` with whatever name you want for your app.
- You may notice the `setup.sh` file in this repo. This just essentially replaces whatever is in the `lib/` directory with any new dependencies added or removed from the `requirements.txt` file. This can be run with `source setup.sh`.


## Sources
- [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [How to Host Multiple Flask Apps on Digital Ocean](https://www.reddit.com/r/flask/comments/2v39a3/how_to_host_multiple_flask_apps_on_digital_ocean/)