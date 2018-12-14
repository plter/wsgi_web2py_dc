# wsgi_web2py_dc
WSGI with web2py docker container 

# How to use  

1. `docker run -it -p 80:80 -p 443:443 xtiqin/wsgi_web2py`
1. `/opt/lampp/lampp start`  
1. Then you can visit `http://127.0.0.1`  

# Config docker image/container for web2py and mod_wsgi 

**If you want to config docker image by yourself,please see this section.**   

## Step 1. Install docker ce 

* Register a docker account on [https://www.docker.com/](https://www.docker.com/) 
* Download docker ce
* Install docker ce follow [https://docs.docker.com/install/](https://docs.docker.com/install/)

## Step 2. Download and run ubuntu image 

We set up environment based on ubuntu 18.04.  
* `docker pull ubuntu:18.04` 
* `docker run -it -p 80:80 -p 443:443 ubuntu:18.04`  

Then we will login to the running docker container.

* Run `apt update` to update repos.
* Run `apt install tzdata` to setup timezone. 
* Run `apt install gcc` to install gcc
  
## Step 3. Download and install xampp 

* Download latest linux 64 bit version from [https://www.apachefriends.org/download.html](https://www.apachefriends.org/download.html)
* Install xampp (Remember to install `XAMPP Developer Files`)

After successfully installed, xampp will be located in path `/opt/lampp` 

* Use `/opt/lampp/lampp start` to start all lampp services.
* Use `/opt/lampp/lampp stop` to stop all lampp services.
* Use `/opt/lampp/lampp restart` to restart all services.
* Use `/opt/lampp/lampp startapache` to start apache only.   

For better using xampp, we need to install `net-tools` with command `apt install net-tools`  

Then we should link `apxs` and `perl`   
* `ln -s /opt/lampp/bin/apxs /bin/apxs`  
* `ln -s /opt/lampp/bin/perl /bin/perl`  

## Step 4. Install python2 and it's dev package

* `apt install python`  
* `apt install python-dev`  

## Step 5. Install pip 

Following [https://pip.pypa.io/en/stable/installing/](https://pip.pypa.io/en/stable/installing/) 

## Step 6. Download web2py 

* Download web2py source code from [http://web2py.com/init/default/download](http://web2py.com/init/default/download)  
* Extract the package to `/opt/lampp/wsgiroot/web2py`  
* `chown -R www-data:www-data /opt/lampp/wsgiroot/web2py/`
* `cp /opt/lampp/wsgiroot/web2py/handlers/wsgihandler.py /opt/lampp/wsgiroot/web2py/`  
* `chmod +x /opt/lampp/wsgiroot/web2py/wsgihandler.py`  
* `chmod -R 755 /opt/lampp/wsgiroot/web2py/`

## Step 7. Install and config mod_wsgi 

Following [https://pypi.org/project/mod_wsgi/](https://pypi.org/project/mod_wsgi/)  

* `pip install mod_wsgi` 
* Make a file in `/opt/lampp/etc/extra/httpd-wsgi.conf`  
* Run `mod_wsgi-express module-config` ,you will see some message output like this  
```apacheconfig
LoadModule wsgi_module "/usr/local/lib/python2.7/dist-packages/mod_wsgi/server/mod_wsgi-py27.so"
WSGIPythonHome "/usr"
```  
* Then we can append the following text to file `/opt/lampp/etc/extra/httpd-wsgi.conf` 
```apacheconfig
LoadModule wsgi_module "/usr/local/lib/python2.7/dist-packages/mod_wsgi/server/mod_wsgi-py27.so"
WSGIPythonHome "/usr"

WSGIDaemonProcess web2py user=www-data group=www-data home=/opt/lampp/wsgiroot/web2py

<VirtualHost *:80>
  WSGIProcessGroup web2py
  WSGIScriptAlias / /opt/lampp/wsgiroot/web2py/wsgihandler.py
  WSGIPassAuthorization On

  <Directory /opt/lampp/wsgiroot/web2py>
    AllowOverride None
    Require all denied
    <Files wsgihandler.py>
      Require all granted
    </Files>
  </Directory>

  AliasMatch ^/([^/]+)/static/(?:_[\d]+.[\d]+.[\d]+/)?(.*) /opt/lampp/wsgiroot/web2py/applications/$1/static/$2

  <Directory /opt/lampp/wsgiroot/web2py/applications/*/static/>
    Options -Indexes
    ExpiresActive On
    ExpiresDefault "access plus 1 hour"
    Require all granted
  </Directory>

  CustomLog /opt/lampp/logs/web2py_access.log common
  ErrorLog /opt/lampp/logs/web2py_error.log
</VirtualHost>

<VirtualHost *:443>
  SSLEngine on
  SSLCertificateFile /opt/lampp/etc/ssl.crt/server.crt
  SSLCertificateKeyFile /opt/lampp/etc/ssl.key/server.key

  WSGIProcessGroup web2py
  WSGIScriptAlias / /opt/lampp/wsgiroot/web2py/wsgihandler.py
  WSGIPassAuthorization On

  <Directory /opt/lampp/wsgiroot/web2py>
    AllowOverride None
    Require all denied
    <Files wsgihandler.py>
      Require all granted
    </Files>
  </Directory>

  AliasMatch ^/([^/]+)/static/(?:_[\d]+.[\d]+.[\d]+/)?(.*) /opt/lampp/wsgiroot/web2py/applications/$1/static/$2

  <Directory /opt/lampp/wsgiroot/web2py/applications/*/static/>
    Options -Indexes
    ExpiresActive On
    ExpiresDefault "access plus 1 hour"
    Require all granted
  </Directory>

  CustomLog /opt/lampp/logs/web2py_ssl-access.log common
  ErrorLog /opt/lampp/logs/web2py_error.log
</VirtualHost>

Alias /dashboard /opt/lampp/htdocs/dashboard

```
* Add line `Include etc/extra/httpd-wsgi.conf` to file `/opt/lampp/etc/httpd.conf` to add wsgi config file 
