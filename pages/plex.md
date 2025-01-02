---
title: Plex
---

# Plex
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Install

Install the package

```sh
$ sudo dpkg -i plexmediaserver_0.9.8.18.290-11b7fdd_amd64.deb
```

The default manager URL is http://127.0.0.1:32400/web

Change this manager URL using apache

* Install apache

```sh
$ sudo apt-get install apache2
```

* import modules

```sh
$ sudo a2enmod
```

* Configure `/etc/apache2/sites-available/000-default.conf`

```xml
<VirtualHost *:80>
  <Proxy *>
    Order deny,allow
    Allow from all
  </Proxy>

  ProxyRequests Off
  ProxyPreserveHost On

  <Location />
    ProxyPass http://localhost:32400/
    ProxyPassReverse http://localhost:32400/
  </Location>

  <Location /gui/>
    ProxyPass http://localhost:8080/gui/
    ProxyPassReverse http://localhost:8080/gui/
  </Location>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

* Restart apache

```sh
$ sudo service apache2 restart
```

Set password

* Add user and password

```sh
$ sudo htpasswd -bc /etc/apache2/htpasswd <username> <password>
```

* Configure `/etc/apache2/sites-available/000-default.conf`

```xml
<VirtualHost *:80>
  <Proxy *>
    Order deny,allow
    Allow from all
  </Proxy>

  ProxyRequests Off
  ProxyPreserveHost On

  <Location />
    AuthType Basic
    AuthName "Restricted area"
    AuthUserFile /etc/apache2/htpasswd
    Require valid-user

    ProxyPass http://localhost:32400/
    ProxyPassReverse http://localhost:32400/
  </Location>

  <Location /gui/>
    ProxyPass http://localhost:8080/gui/
    ProxyPassReverse http://localhost:8080/gui/
  </Location>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

* Restart apache

```sh
$ sudo service apache2 restart
```

Set SSL

* Generate a self-signed certificate

```ssh
sudo openssl req -newkey rsa:2048 -x509 -sha256 -days 365 -nodes -out /etc/apache2/apache.pem -keyout /etc/apache2/apa
```

* Configure `/etc/apache2/sites-available/000-default.conf`

```xml
<VirtualHost *:443>
  <Proxy *>
    Order deny,allow
    Allow from all
  </Proxy>

  SSLEngine On
  SSLProxyEngine On
  SSLCertificateFile /etc/apache2/apache.pem
  SSLCertificateKeyFile /etc/apache2/apache.key

  ProxyRequests Off
  ProxyPreserveHost On

  <Location />
    AuthType Basic
    AuthName "Restricted area"
    AuthUserFile /etc/apache2/htpasswd
    Require valid-user

    ProxyPass http://localhost:32400/
    ProxyPassReverse http://localhost:32400/
  </Location>

  <Location /gui/>
    ProxyPass http://localhost:8080/gui/
    ProxyPassReverse http://localhost:8080/gui/
  </Location>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

* Restart apache

```sh
$ sudo service apache2 restart
```
