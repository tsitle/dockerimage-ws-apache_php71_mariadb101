# Apache2 + PHP 7.1 + MariaDB 10.1 Client Docker Image for AARCH64, ARMv7l, X86 and X64

For hosting PHP powered websites.

## Inheritance and added packages
- Docker Image **tsle/ws-apache-base** (see [https://github.com/tsitle/dockerimage-ws-apache\_base](https://github.com/tsitle/dockerimage-ws-apache_base))
	- PHP 7.1 (CLI + FPM)
	- PHP packages (see below)
	- MariaDB Client 10.1 (^= MySQL 5.7)
	- php-pear
	- xml-core

## PHP Packages included
- bcmath
- cli
- common
- curl
- fpm
- gd
- imagick
- imap
- json
- mbstring
- mcrypt
- mysql
- opcache
- readline
- sqlite3
- xdebug
- xml
- zip

## Webserver TCP Port
The webserver is listening only on TCP port 80 by default.

## Docker Container usage
See the related GitHub repository [https://github.com/tsitle/dockercontainer-ws-apache\_php71\_mariadb101](https://github.com/tsitle/dockercontainer-ws-apache_php71_mariadb101)

## Docker Container configuration
From **tsle/ws-apache-base**:

- CF\_PROJ\_PRIMARY\_FQDN [string]: FQDN for website (e.g. "mywebsite.localhost")
- CF\_SET\_OWNER\_AND\_PERMS\_WEBROOT [bool]: Recursively chown and chmod CF\_WEBROOT?
- CF\_WWWDATA\_USER\_ID [int]: User-ID for www-data
- CF\_WWWDATA\_GROUP\_ID [int]: Group-ID for www-data
- CF\_ENABLE\_CRON [bool]: Enable cron service?
- CF\_LANG [string]: Language to use (en\_EN.UTF-8 or de\_DE.UTF-8)
- CF\_TIMEZONE [string]: Timezone (e.g. 'Europe/Berlin')

From this image:

- CF\_WWWFPM\_USER\_ID [int]: User-ID for wwwphpfpm
- CF\_WWWFPM\_GROUP\_ID [int]: Group-ID for wwwphpfpm

## Using cron
You'll need to create the crontab file `./mpcron/wwwphpfpm` and then add some task to the file:

```
# the following command will be executed as 'wwwphpfpm'
* *    *   *   *     cd /var/www/html/; tar cf backup.tar site-html/> /dev/null 2>&1
```

Instead of the username `wwwphpfpm` you could also use `root`.

Now you could enable cron in your docker-compose.yaml file like this:

```
version: '3.5'
services:
  apache:
    image: "ws-apache-base-<ARCH>:<VERSION>"
    ports:
      - "80:80"
    volumes:
      - "$PWD/mpweb:/var/www/html"
      - "$PWD/mpcron/wwwphpfpm:/var/spool/cron/crontabs/wwwphpfpm"
    environment:
      - CF_PROJ_PRIMARY_FQDN=example-host.localhost
      - CF_WWWFPM_USER_ID=<YOUR_UID>
      - CF_WWWFPM_GROUP_ID=<YOUR_GID>
      - CF_SET_OWNER_AND_PERMS_WEBROOT=false
      - CF_ENABLE_CRON=true
      - CF_LANG=de_DE.UTF-8
      - CF_TIMEZONE=Europe/Berlin
    restart: unless-stopped
    stdin_open: false
    tty: false
```

## Enabling the PHP Module XDebug
The PHP Module 'xdebug' is disabled by default.  
To enable it you'll need to follow these steps from within a Bash shell:  
(replace "DOCKERCONTAINER" with your Docker Container's name)

1. Start the Docker Container
2. If your debugger (e.g. IntelliJ) is running on a different machine then
	you'll need to replace the default hostname "host" with your machine's hostname.  
	Edit XDebug configuration:  
	```
	$ docker exec -it DOCKERCONTAINER nano /etc/php/7.1/mods-available/xdebug.ini
	```  
	  
	```
	xdebug.remote_host="host"
	```  
	When done editing hit CTRL-X, then "J" and hit ENTER
3. Now enable the PHP Module:  
	```  
	$ docker exec -it DOCKERCONTAINER phpenmod xdebug
	```  
	```  
	$ docker exec -it DOCKERCONTAINER service php7.1-fpm restart
	```

## Disabling the PHP Module XDebug
```  
$ docker exec -it DOCKERCONTAINER phpdismod xdebug
```  
```  
$ docker exec -it DOCKERCONTAINER service php7.1-fpm restart
```
