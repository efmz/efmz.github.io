---
layout: post
title:  "Using Mailcatcher In Your PHP Project"
date:   2018-02-25 11:32:49 +0000
categories: howto
---
## Mailcatcher - a fake SMTP server

Firstly, why would you need a fake SMTP server? The primary reason would be to test your application's outgoing emails without accidentally spamming real e-mail addresses (and real customers!). It's also very useful to see the content and formatting of outgoing e-mails sent from your app or website.

You may have heard of services like mailtrap.io. But if you're iffy about sending your test emails to a 3rd party service, and want a solution that is locally hosted and FREE, [Mailcatcher](https://mailcatcher.me) is your answer. It's simple to setup and works like a dream. 


## Setting up Mailcatcher in a subdomain

This is a guide on how to install and setup Mailcatcher in Centos 7. By default, after starting the Mailcatcher service, you would access Mailcatcher 'inbox' via http://localhost:1080/. What if you wanted other developers in your team to access this too in e.g a staging server? Something like http://mailcatcher.mydevsite.io perhaps?

### 1. Install Ruby and Ruby Gem

In your dev/staging machine, make sure you have Ruby `gem` installed. If you don't have this already:

`sudo yum install -y ruby ruby-devel rubygems`

Make sure to install development tools first:

`sudo yum -y install git-core zlib zlib-devel gcc-c++ patch readline readline-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison curl sqlite-devel`

### 2. Install Mailcatcher

`gem install mailcatcher`

And wait for installation to finish. This can take a while.

### 3. Run Mailcatcher as a daemon

Start Mailcatcher as a daemon and bind to all network interfaces:

`mailcatcher --ip=0.0.0.0`

Go to http://localhost:1080 to access the Mailcatcher web interface.

However, this will not be accessible through http://mailcatcher.devsite.io:1080 -so you have to set this up in a subdomain, by **using Apache as a reverse proxy**

### Access Mailcatcher from a subdomain using Apache Vhosts

Most other tutorials for reverse proxy involve Nginx, but we don't want the fuss and bother of installing Nginx just
for this purpose.

*Source: https://www.digitalocean.com/community/tutorials/how-to-use-apache-as-a-reverse-proxy-with-mod_proxy-on-centos-7*

**The important steps from the above tutorial:**

1. We need to make sure these Apache modules are installed:
`mod_proxy, mod_proxy_http, mod_proxy_balancer, mod_lbmethod_byrequests`

To check:

`httpd -M`

Output should include:

    . . .
     proxy_module (shared)
    . . .
     lbmethod_byrequests_module (shared)
    . . .
     proxy_balancer_module (shared)
    . . .
     proxy_http_module (shared)
    . . .

If any of these modules are missing, read the tutorial to install them

2. Edit the the relevant vhosts file in /etc/httpd/conf.d/ to include this:

        <VirtualHost *:80>
                ServerName mailcatcher.mailcatcher.devsite.io
                ProxyPreserveHost On
                ProxyPass / http://127.0.0.1:1080/
                ProxyPassReverse / http://127.0.0.1:1080/
        </VirtualHost>

3. Restart apache `sudo apachectl restart`

4. Mailcatcher should now be accessible at http://mailcatcher.mailcatcher.devsite.io/


## Using Mailcatcher in your PHP app

### Using PHP mail()

If we're using the generic PHP `mail()` function, we need to set the sendmail config in `/etc/php.ini`

Find this line, comment it out like so:

`;sendmail_path = /usr/sbin/sendmail -t -i`

We're commenting out and not removing it because we'll want to use sendmail again at some point.

And add this line:

`sendmail_path = /usr/bin/env catchmail -f some@from.address`

The above is so that the PHP `mail()` function uses Mailcatcher's `catchmail` instead of `sendmail`

Then restart Apache `sudo apachectl restart`

That's it! Send out emails from your PHP app, and you will see it Mailcatcher's web interface.

---
**NOTE:** In my case, this seems to only work when running tests through `phpunit` but doesn't work when actually using it from within
the website (e.g from client Control Panel), so we will need to keep `mail()` using `sendmail`, and use another PHP library to send mail via SMTP
 e.g PHPMailer
---

### Using PHPMailer

When using PHPMailer or any external PHP library for sending mail through SMTP, use the details below:

SMTP host: localhost
SMTP port: 1025
No auth/username/password
Use TLS

**The above method, using PHPMailer, works in my case for both `phpunit` and from testing in the browser**





