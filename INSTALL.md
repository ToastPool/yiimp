# Ubuntu 16.04 Installation

## Get the webserver up and running

install stratum and webserver dependencies
---
> sudo apt install php php-curl libcurl4-openssl-dev php-mysql php-memcache mysql-server libmysqlclient-dev nginx memcached screen libldap2-dev libidn11-dev librtmp-dev libkrb5-dev sendmail

set up nginx
---
First, make sure there you have no configs already specified as "default_server" or "server_name _".
Then, create /etc/nginx/sites-enabled/pool.conf:

	server {
	        listen 80 default_server;
	        listen [::]:80 default_server;

	        root /var/www/web;
	        index index.html index.htm;

	        server_name _;

	        location / {
	            try_files $uri @rewrite;
	        }

	        location @rewrite {
	            rewrite ^/(.*)$ /index.php?r=$1;
	        }

	        location ~ \.php$ {
	            include snippets/fastcgi-php.conf;
	            fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
	        }
	}

restart nginx and verify that it's running
---
> sudo service nginx restart
> sudo service nginx status

set up sql database
---
	mysql>  CREATE DATABASE pool;

set up sql accounts for users php and yiimp
---
	mysql> CREATE USER 'php'@'localhost' IDENTIFIED BY 'password';
	mysql> GRANT ALL PRIVILEGES ON *.* TO 'php'@'localhost' WITH GRANT OPTION;
	mysql> CREATE USER 'yiimp'@'localhost' IDENTIFIED BY 'password';
	mysql> GRANT ALL PRIVILEGES ON *.* TO 'yiimp'@'localhost' WITH GRANT OPTION;

load initial sql data
---
> cd sql
> ./load_sql.sh
> cd ..

set up stratum folder
---
> mkdir /var/stratum
> mkdir /var/stratum/config
> cp -rf stratum/config /var/stratum/
> cp stratum/config/foo.conf /var/stratum
> sudo cp stratum/stratum /var/stratum/
> sudo cp blocknotify/blocknotify /var/stratum/

setup web folder
---
> sudo cp web/keys.sample.php /var/www/web/keys.php

copy over
---
> sudo cp -rf web/ /var/www/web

create sql folder
---
> mkdir /var/www/sql

then configure the keys file
---
> sudo vim /var/www/web/keys.php

then configure the web app
---
> sudo mv /var/web/serverconfig.sample.php /var/web/serverconfig.php
> sudo vim /var/web/serverconfig.php
> ... need to change YAAMP_DBNAME, YAAMP_DBUSER, and YAAMP_DBPASSWORD
> ... also set YAAMP_RENTAL to false, YAAMP_SITE_NAME to preferred name, and configure YAAMP_ADMIN_EMAIL and YAAMP_ADMIN_IP

make sure everything works
---
> cd bin
> ./yiimp checkup

run scripts in screen
---
> cd /var/www/web
> screen
> ./main.sh
> ... switch screen ...
> ./loop2.sh
> ... switch screen ...
> ./block.sh

Now your webserver should be running! Check it in your web browser

## Get the stratum server up and running

set up Go (must be at least 1.10)
---
> apt install golang-1.10

add the following to your ~/.bashrc

	export GOPATH=$HOME/.go
	export PATH=$PATH:/usr/lib/go-1.10/bin:$GOPATH/bin

> source ~/.bashrc

build Sia stratum server
---
> go get -u github.com/ToastPool/Sia/...
> mv ~/.go/src/github.com/ToastPool/Sia ~/.go/src/github.com/NebulousLabs/Sia
> cd ~/.go/src/github.com/NebulousLabs/Sia
> make dependencies
> make release

configure and run Sia stratum server
---
> mkdir ~/siad_data && cd ~/siad_data
> cp ~/.go/src/github.com/NebulousLabs/Sia/sampleconfigs/sia.yml ~/siad_data/
> vim sia.yml
> siad -M cgtwp

# For docker users

copy and change docker env
---
> cd yiimplara
> cp env-example .env
> vi .env (change mysql part)

create rpc external network
---
> docker network create rpc_net

run docker
---
> cd yiimplara
> make up

run mysql container(optional, might be conflict with local mysql port)
---
> cd lara
> docker-compose up -d mysql

load mysql data to mysql container database
---
> mysql -uroot -proot < sql/{newestsqlfile}

copy and change web configs
---
> cd web
> cp keys.sample.php keys.php
> cp serverconfig.sample.php serverconfig.php
> (change both config file)

open browser and checkout php-fpm/nginx/web/php-runtime logs
---
> docker-compose logs -f php-fpm
> docker-compose logs -f nginx
> tail -f ./logs/web/debug.log
> tail -f ./web/yaamp/runtime/application.log

go into php-fpm container (could run php codes)
---
> make in
