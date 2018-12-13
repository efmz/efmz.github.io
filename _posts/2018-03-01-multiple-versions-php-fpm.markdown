---
layout: post
title:  "Running Multiple Versions of PHP with PHP-FPM"
date:   2018-03-01 20:00:00 +0000
categories: howto,php,legacy
---

# Running Multiple Versions of PHP With PHP-FPM


## Introduction

Lets say we need both PHP 5.6 and PHP 7 to run in parallel - the older version for your legacy sites and the latest version for sites using a modern framework like Laravel.
So one subdomain would use PHP 5.6, and another subdomain will use PHP 7. We need **PHP-FPM**.

The following steps are based on this article:
https://blog.remirepo.net/post/2016/04/16/My-PHP-Workstation

## Assumptions

* The'default' version of PHP currently in use is PHP 5.6, and we want to install PHP 7.1
* We have already added the REMI repo (remi-safe)

## Steps

1) Install *Software Collections* - to enable running multiple versions in parallel

`sudo yum install centos-release-scl`

2) In order to see what PHP 7.1 modules we need to install, we need to list the php packages we have installed already

`yum list installed | grep php`

You should see something like this:

    php56w.x86_64                      5.6.31-2.w7                        @webtatic
    php56w-bcmath.x86_64               5.6.31-2.w7                        @webtatic
    php56w-cli.x86_64                  5.6.31-2.w7                        @webtatic
    php56w-common.x86_64               5.6.31-2.w7                        @webtatic
    php56w-gd.x86_64                   5.6.31-2.w7                        @webtatic
    php56w-mbstring.x86_64             5.6.31-2.w7                        @webtatic
    php56w-mysql.x86_64                5.6.31-2.w7                        @webtatic
    php56w-opcache.x86_64              5.6.31-2.w7                        @webtatic
    php56w-pdo.x86_64                  5.6.31-2.w7                        @webtatic
    php56w-process.x86_64              5.6.31-2.w7                        @webtatic
    php56w-tidy.x86_64                 5.6.31-2.w7                        @webtatic
    php56w-xml.x86_64                  5.6.31-2.w7                        @webtatic


3) Now install PHP 7.1 modules

`sudo yum install php71 php71-php-fpm php71-php-mbstring php71-gd php71-mysql php71-php-pdo php71-php-mysqlnd php71-php-xml`

*We need all of the above for Laravel*

4) Configure PHP-FPM

`sudo vi /etc/opt/remi/php71/php-fpm.d/www.conf`

Look for the 'listen' config, and change to: (the port we are using is 9071, but it can be anything e.g 9000)

    listen = 127.0.0.1:9071
    pm = ondemand

5) Configure SELinux context for this port

`sudo semanage port -a -t http_port_t -p tcp 9071`

6) Start the PHP-FPM service, and set it up so that it does so on reboot too:

`sudo systemctl start php71-php-fpm`

`sudo systemctl enable php71-php-fpm`

7) Now it's time to configure it for your sites. In the Apache vhosts config file:

`sudo vim /etc/httpd/conf.d/vhost.conf`

Add this for the subdomain that you want to run PHP 7.1

    <VirtualHost *:80>
            ServerAdmin admin@example.com
            ServerName php71site.example.com
            DocumentRoot /var/www/html/php71site/public_html

            <FilesMatch \.php$>
                SetHandler "proxy:fcgi://127.0.0.1:9071"
            </FilesMatch>

    </VirtualHost>

8) Check everything is working by doing a `phpinfo()` in a `test.php` file within the subdomain's document root

You should see this when navigating to http://php71site.example.com/test.php:

PHP Version 7.1.11
Server API: 	FPM/FastCGI

## Troubleshooting

### Laravel: 500 Internal Server Error

As long as you've tested with `test.php` as above, this should be an application error.

Best thing to do would be to uncomment any custom Exception packages in `bootstrap/app.php` so that the full error message shows.

###Missing PHP modules

If you find that the errors are caused by missing PHP 7.1 module(s), install them, then restart php-fpm (and Apache too just for good measure?)

e.g

    sudo yum install php71-php-pdo php71-php-mysqlnd

    sudo systemctl restart php71-php-fpm

    sudo apachectl restart

## PHP 7 from command line

When doing things like `composer update` where some dependencies require PHP 7, we need to switch to PHP 7 temporarily

`module load php71`

Then run your update

`composer update`

Install whatever missing PHP modules you still need and unload PHP 7.1 again

`module unload php71`




