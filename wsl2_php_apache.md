# WSL2 PHP & Apache
I ran into issues when I was setting up PHP and Apache on WSL2, because the Ubuntu distro you can use for WSL2 works a bit different than the standard.

## Prerequisites
I assume WSL2 is up and running with Ubuntu. If not, you can do that here: https://docs.microsoft.com/en-us/windows/wsl/install-win10

## Install Apache2
```
#!/usr/bin/env bash

# Install Apache
sudo apt install --yes apache2;

# Enable modules
sudo a2enmod vhost_alias
sudo a2enmod rewrite;
sudo a2enmod headers;

# Restart Apache
sudo service apache2 restart;
```

## Install PHP
For this we take 7.4
```
# Add repository
$ sudo add-apt-repository --yes ppa:ondrej/php;
$ sudo apt update;

# Install PHP 7.4
$ sudo apt install --yes \
    php7.4 \
    php7.4-amqp \
    php7.4-bcmath \
    php7.4-cli \
    php7.4-curl \
    php7.4-dev \
    php7.4-gd \
    php7.4-imap \
    php7.4-intl \
    php7.4-ldap \
    php7.4-mbstring \
    php7.4-memcached \
    php7.4-mysql \
    php7.4-opcache \
    php7.4-pspell \
    php7.4-soap \
    php7.4-sqlite3 \
    php7.4-xml \
    php7.4-zip \
    php-igbinary \
    php-msgpack \
    libapache2-mod-php7.4;

$ sudo a2enmod php7.4;

# Restart Apache
$ sudo service apache2 restart;

# Configure PHP 7.2 as default version
$ sudo update-alternatives --set php /usr/bin/php7.4;
$ sudo update-alternatives --set php-config /usr/bin/php-config7.4;
$ sudo update-alternatives --set phpize /usr/bin/phpize7.4;
```

## Apache VHOST file
```
# Fix ownership
# Fill in your own user and group
sudo chown --recursive "${USER}:${USER_GROUP}" /var/www;

# Remove 'default' website
$ rm --recursive --force /var/www/html;
```

New file: /etc/apache2/sites-available/www.conf

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName wsl.local
    ServerAlias *.wsl.local

    VirtualDocumentRoot /var/www/%1

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory "/var/www">
        AllowOverride All
    </Directory>

    RewriteEngine On
    RewriteMap lowercase int:tolower
</VirtualHost>
```
To enable: 
```
$ sudo a2enmod www.conf
```

## Apache configuration
Add to /etc/apache2/apache2.conf

```
<FilesMatch \.php$>
SetHandler application/x-httpd-php
​</FilesMatch>
```
```
$ sudo a2dismod mpm_event && sudo a2enmod mpm_prefork && sudo a2enmod php7.4
```
Restart apache.

## Using Symfony
I noticed that if you use Symfony, the requests are not properly redirected.
Add the following .htaccess in the root of your application so that requests are redirected:
```
RewriteEngine on
RewriteRule ^(.*)$ /public/$1 [L]
```
Also make sure the apache pack is installed:
```
$ composer require symfony/apache-pack
```

## Hosts file
Check your ip address in your WSL2:
```
$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.31.168.33  netmask 255.255.240.0  broadcast 172.31.175.255
        inet6 fe80::215:5dff:feaf:5a67  prefixlen 64  scopeid 0x20<link>
        ether 00:15:5d:af:5a:67  txqueuelen 1000  (Ethernet)
        RX packets 745318  bytes 774612934 (774.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 345839  bytes 45889235 (45.8 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
So mine is 172.31.168.33, yours is different.

Add that to your hosts file in Windows C:\Windows\System32\drivers\etc\hosts:
```
172.31.168.33 yourapp.wsl.local
```

Also add to the hosts file on your WSL2:
```
127.0.0.1 yourapp.wsl.local
```

Now you should be able to browse to http://yourapp.wsl.local