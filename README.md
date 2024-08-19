Guide to install a LAMP system on on your **archlinux** system and serve php-based database applications.    
***LAMP*** stands for a **Linux** system with **Apache** (webserver), **MariaDB** (database) and **PHP** (programming language). In this guide we will also install **PhpMyAdmin** (database admin GUI) to easily manage the SQL tables.

# Apache

![apache logo](https://upload.wikimedia.org/wikipedia/commons/thumb/1/10/Apache_HTTP_server_logo_%282019-present%29.svg/300px-Apache_HTTP_server_logo_%282019-present%29.svg.png)

*The Apache HTTP Server is an open-source and free product of the Apache Software Foundation and one of the most widely used web servers on the Internet. In addition to factors such as performance, expandability, security, freedom from license costs and support from a very large community, its long-term availability for a wide variety of operating systems is one of the reasons for its widespread use; it is most frequently used as a LAMP system.*

## Install packages

`pacman -S apache`


## Config Apache

The main **Apache configuration file** is located at `/etc/httpd/conf/httpd.conf`
The default **Document Root** is `/srv/http`. The webserver will serve all files that are located under this folder. 

> On linux lammp installation the `httpd.conf` is located at `/opt/lampp/etc/httpd.conf`    
> The default Document root is: `/opt/lampp/htdocs`

Make sure to restart apache after a configuration to see it's effects:

```shell
systemctl start   httpd.service
systemctl stop    httpd.service
systemctl restart httpd.service
systemctl enable  httpd.service  #autostart service on boot
```


### Config file for your webapp
To tell apache where your app is located and with which alias it can be accessed create a new config file: `/etc/httpd/conf/extra/<YOUR_APP>.conf`

```conf
Alias /<YOUR_ALIAS> "path/to/application"
<Directory "path/to/application">
    DirectoryIndex index.html
    AllowOverride All
    Options FollowSymlinks
    Require all granted
</Directory>
```

Now we have to tell apache to use this new config file. Add the following line to your `/etc/httpd/conf/httpd.conf`
```shell
Include conf/extra/<YOUR_APP>.conf
```

### Permissions
This was the main part, but mostly the apache daemon does not have access rights to the specified folder in your filesystem. The result is a **403 Error**.

To fix this, you can give every user full access to your project folder
> **⚠️ Attention:** make sure if that makes sense in your use case:
```shell
chmod -R 777 /path/to/project/
```

Make sure, that apache has also has access to the folder that contains your project (Read access is enough here)
E.g. if your project is located under `/home/username/project`use:

```shell
chmod 711 /home/username/
```


# PHP

![php logo](https://upload.wikimedia.org/wikipedia/commons/thumb/2/27/PHP-logo.svg/300px-PHP-logo.svg.png)   
*PHP (for "PHP: Hypertext Preprocessor") is a scripting language with a syntax based on C and Perl, which is mainly used to create dynamic websites or web applications. PHP is distributed as free software under the PHP license. PHP is characterized by broad database support and Internet protocol integration as well as the availability of numerous function libraries.*

## Install Packages

`pacman -S php php-apache`

## Config PHP

The main PHP configuration file is well-documented and located at `/etc/php/php.ini`

### Error reporting
For testing purposes it might be very helpful to enable PHP's error messages:

If you want to display errors in browser set in the **php.ini**: `display_errors = On`   
You can also enable error reporting in `.htaccess` file: `php_flag display_errors On`

> **⚠️ Attention:** you should not display errors in productive environments.


### Sql Module
If you want to use MySql operations in your PHP scripts to connect to a MySQL database, uncomment these lines in **php.ini**:
```ini
extension=mysqli        # for using the mysqli commands
extension=pdo_mysql     # for using PDO (Php Database Objects API)
```

### Sessions
If you want to use sessions go to the `[Session]` section in php.ini   
Uncomment the line:
```ini
session.save_path = "/tmp"
```

### Sendmail
Configure how PHP sends mails when usign the `mail()` function at the `[mail function]` section in php.ini   
For testing you can simulate sending mails to a logfile:
```ini
sendmail_path = 'cat >> /your/path/sendmail.log'
mail.log = "/your/path/sendmail.log"
```
> Make sure Apache has write permissions for that file

### Let Apache use PHP

By default Apache would just serve your php files as plain text files. To make apache execute your php scripts follow these steps:

#### mod_mpm

Make sure that the **mpm_event module** is **disabled** and the **mpm_prefork** module is **enabled**

In your httpd.conf
```conf
#LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
```

#### Enable PHP module

Add to your httpd.conf:
```conf
LoadModule php_module modules/libphp.so
AddHandler php-script .php
Include	conf/extra/php_module.conf
```

# MariaDB

![mariadb logo](https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/MariaDB_colour_logo.svg/440px-MariaDB_colour_logo.svg.png)   
*MariaDB is a free, relational open source database management system that was created by a fork from MySQL. The project was initiated by MySQL's former main developer Michael Widenius, who also developed the storage engine Aria, on which MariaDB was originally based.*

## Install Packages

`pacman -S mariadb`

## Config MariaDB

Run `mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql`

Start the mariadb service: `systemctl enable --now mariadb`

### Create user and Database

#### Connect to mysql server from your root systemuser via socket (no password required)

````shell
mysql --protocol=socket #run this command as root (e.g. prefixed with sudo)
````

#### Create a new user
````shell
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
````

#### Create new database and grant privilleges
````shell
CREATE DATABASE `db_name`;
GRANT ALL PRIVILEGES ON `db_name` . * TO 'username'@'localhost';
````



# PhpMyAdmin

![pma logo](https://upload.wikimedia.org/wikipedia/de/thumb/e/ef/PhpMyAdmin-Logo.svg/350px-PhpMyAdmin-Logo.svg.png)   
*phpMyAdmin (PMA for short) is a free web application for the administration of MySQL databases and MariaDB. The software is implemented in PHP, hence the name phpMyAdmin. Most functions can be executed without writing SQL statements, such as listing data records, creating/deleting tables, adding columns, creating/deleting databases and managing users.*

## Install Packages

`pacman -S phpmyadmin`

## Config PhpMyAdmin

Create the Apache configuration file:

`/etc/httpd/conf/extra/phpmyadmin.conf`
```txt
Alias /phpmyadmin "/usr/share/webapps/phpMyAdmin"
<Directory "/usr/share/webapps/phpMyAdmin">
    DirectoryIndex index.php
    AllowOverride All
    Options FollowSymlinks
    Require all granted
</Directory>
```

And include it in `/etc/httpd/conf/httpd.conf`:

```txt
# phpMyAdmin configuration
Include conf/extra/phpmyadmin.conf
```

You can now access the PhpMyAdmin webinterface at: `http://localhost/phpmyadmin`


# Use your application

Make sure to restart the apache daemon after your configurations: `systemctl restart httpd`
Open your browser and go to: `localhost/your-app`