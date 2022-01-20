# Installing LAMP stack on Windows 10/11 using WSL2

The following guide will help to install the following components in Windows 10/11 using WSL2. And also the guide going through to install 2 versions of PHP with help of PHP-FPM and therefore we can easily choose the version of PHP using `.htaccess`.

1. Linux - The operating system
2. Apache - The Web Server
3. MySQL - The Database
4. PHP - The programming language

> Ubuntu distro used as Linux operating system.

## Enable WSL2 and Install Ubuntu

`wsl --install`

> Above comand should install Ubuntu as default distro, if not please follow the following link: <https://docs.microsoft.com/en-us/windows/wsl/setup/environment>

## Upgrade Ubuntu to latest (optional)

`sudo do-release-upgrade -d`

## Install Apache

`sudo apt update && sudo apt install apache2`

## Add repositories to install PHP

`sudo apt update`

`sudo apt install libapache2-mod-fcgid`

`sudo apt install software-properties-common`

### Add ondrej/php

`sudo LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php && sudo apt update`

OR

### Add ondrej/apache2

`sudo LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/apache2 && sudo apt update`

> add-apt-repository is broken with non-UTF-8 locales, to fix this added LC_ALL=C UTF-8 to the above command

## Install MySQL

`sudo apt install mysql-server`

After install restart `mysql` service

`sudo service mysql restart`

### Secured the mysql installation

`sudo mysql_secure_installation`

Configurations:

 1. Validate password component: n
 2. Add New root password: password
 3. Confirm root password: password
 4. Remove anonymous users: y
 5. Disallow root login remotely: y
 6. Reload privilege tables now: y

### Create mysql user

`sudo mysql -u root`

> Note: Replace `mysql_user` and `mysql_password` based on your interest

mysql command:

```sql
CREATE USER 'mysql_user'@'%' IDENTIFIED BY 'mysql_password';
GRANT ALL PRIVILEGES ON *.* TO 'mysql_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

## Install PHP

PHP 7.4

`sudo apt install php-7.4 php7.4-fpm`

PHP 8.1

`sudo apt install php-7.4 php8.1-fpm`

## Install PHP extensions for both versions

`sudo apt install php7.4-mysql php7.4-curl php7.4-json php7.4-gd php-memcached php7.4-intl php7.4-mbstring php7.4-xml php7.4-zip`

`sudo apt install php8.1-mysql php8.1-curl php8.1-json php8.1-gd php8.1-intl php8.1-mbstring php8.1-xml php8.1-zip`

## Configure apache

`sudo a2enmod actions alias proxy_fcgi fcgid`

Restart apache if required

`sudo service apache2 restart`

### Add PHP services

`sudo a2enconf php7.4-fpm`

`sudo a2enconf php8.1-fpm`

Reload apache if required

`sudo service apache2 reload`

### Enable AllowOverride in order to choose PHP version from .htaccess

`sudo nano /etc/apache2/apache2.conf`

```apacheconf
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

Ctrl + x, y and then Enter to save apache2.conf file and exit

Restart apache

`sudo service apache2 restart`

## Check default PHP version

`php -v`

## Add FilesMatch directive to .htaccess of the applicaiton

This will help to choose the PHP version for the application

For PHP 7.4 (You can skip this if PHP 7.4 default)

```apacheconf
<FilesMatch \.php> 
    # Apache 2.4.10+ can proxy to unix socket
    SetHandler "proxy:unix:/var/run/php/php7.4-fpm.sock|fcgi://localhost/" 
</FilesMatch>
```

For PHP 8.1 (You can skip this if PHP 8.1 default)

```apacheconf
<FilesMatch \.php>
    # Apache 2.4.10+ can proxy to unix socket
    SetHandler "proxy:unix:/var/run/php/php8.1-fpm.sock|fcgi://localhost/"
</FilesMatch>
```

## Install composer

`sudo apt update && sudo apt upgrade`

### Install dependencies first

`sudo apt install wget unzip`

### Download composer setup file

"Navigate to the home folder `cd ~`"

Download the installer from <https://getcomposer.org/installer> and save as composer-setup.php in the current directry

`wget -O composer-setup.php https://getcomposer.org/installer`

### Install composer globally

`sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer`

### Test composer installation

`composer -V`

## Install nodejs

### Install NVM (Node version manager)

> Get the latest version command here: <https://github.com/nvm-sh/nvm#installing-and-updating>
>
> Navigate to the home folder `cd ~`

Download the installer

`wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash`

### Test the NVM installation

The following command should return nvm if not please restart the terminal

`command -v nvm`

### Install latest stable versoin of nodejs

`nvm install --lts`

### Test nodejs

`node -v`

### Test NPM

`npm -v`

## Fix permissions issue when editing files

Add current user into `www-data` group

`sudo usermod -a -G www-data (username)`

Change the group for the working directory

`sudo chgrp www-data /home/(username)/project_dir`

Change the permissions

`sudo chmod -R g+rwxs /home/(username)/project_dir`
