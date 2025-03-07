---
slug: run-php-applications-under-cgi-with-apache-on-ubuntu-9-10-karmic
deprecated: true
author:
  name: Linode
  email: docs@linode.com
description: 'Methods for enabling dynamic content run as individual users with PHP on Ubuntu 9.10 (Karmic).'
keywords: ["php cgi", "php apache", "php scripts", "dynamic apache", "web applications"]
tags: ["web server","apache","ubuntu","php"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
aliases: ['/websites/apache/run-php-applications-under-cgi-with-apache-on-ubuntu-9-10-karmic/','/web-servers/apache/run-php-applications-under-cgi-with-apache-on-ubuntu-9-10-karmic/','/web-servers/apache/php-cgi/ubuntu-9-10-karmic/']
modified: 2012-10-08
modified_by:
  name: Linode
published: 2010-02-17
title: 'Run PHP Applications under CGI with Apache on Ubuntu 9.10 (Karmic)'
relations:
    platform:
        key: php-cgi-apache
        keywords:
            - distribution: Ubuntu 9.10
---



In most cases, we recommend using the `mod_php` module to run PHP scripts with the [Apache HTTP server](/docs/web-servers/apache/). This embeds a PHP interpreter in the web server process and makes running PHP applications easy. The embedded interpreter approach, however, is not without challenges. When the PHP interpreter is embedded in the web server process, PHP scripts are executed by and with the permissions of the web server's user. In smaller deployments, this is perfectly acceptable, but in larger deployments and operations it can create security risks. While Apache's `itk` message passing module (mpm) makes it possible to run Apache processes under user processes in a per-virtual host setup, this is incompatible with the embedded interpreter. The `itk` module is compatible with PHP running as a CGI process.

Additionally, in our experience, `mod_php` is incompatible with the `mod_rails` or Phusion Passenger method of running [Ruby On Rails](/docs/frameworks/). In these cases, if you want to run PHP and Rails applications within a single instance of Apache, you must run PHP scripts as CGI processes using the method outlined below.

Before beginning this guide we assume that you've completed the [Setting Up and Securing a Compute Instance](/docs/guides/set-up-and-secure/). If you are new to Linux server administration, we recommend considering the [beginner's guide](/docs/beginners-guide/), and the article concerning [systems administration basics](/docs/using-linux/administration-basics). If you're interested in learning more about the Apache HTTP server, we encourage you to consider our extensive documentation on [Apache configuration](/docs/web-servers/apache/).

## Enable Universe Repositories

Edit your `/etc/apt/sources.list` file to enable the "universe" repositories by removing the hash symbol in front of the universe lines. The file should resemble the following example:

{{< file "/etc/apt/sources.list" >}}
## main & restricted repositories
deb http://us.archive.ubuntu.com/ubuntu/ karmic main restricted
deb-src http://us.archive.ubuntu.com/ubuntu/ karmic main restricted

deb http://security.ubuntu.com/ubuntu karmic-security main restricted
deb-src http://security.ubuntu.com/ubuntu karmic-security main restricted

## universe repositories
deb http://us.archive.ubuntu.com/ubuntu/ karmic universe
deb-src http://us.archive.ubuntu.com/ubuntu/ karmic universe
deb http://us.archive.ubuntu.com/ubuntu/ karmic-updates universe
deb-src http://us.archive.ubuntu.com/ubuntu/ karmic-updates universe

deb http://security.ubuntu.com/ubuntu karmic-security universe
deb-src http://security.ubuntu.com/ubuntu karmic-security universe

{{< /file >}}


When you have saved this file, issue the following command to refresh your system's package database:

    apt-get update
    apt-get upgrade

We're now ready to begin installing Apache with PHP-CGI.

## Installing Apache and PHP

If you have not already installed the Apache HTTP server, issue the following command to install the packages for Apache:

    apt-get install apache2

You can now [configure virtual hosting](/docs/web-servers/apache/apache-2-web-server-on-ubuntu-9-10-karmic#configure_apache_for_named_based_virtual_hosting) in accordance with the needs of your server. To install the PHP CGI binaries, issue the following command:

    apt-get install php5-cgi

When this process completes, we can configure Apache to hand PHP scripts to the CGI process for rendering these scripts.

## Configure Apache for PHP CGI

In order to set up Apache to use PHP-CGI on Ubuntu systems, you must enable the `mod_actions` module. Issue the following command:

    a2enmod actions

The required directives can be set anywhere in Apache's [configuration tree](/docs/web-servers/apache/configuration/configuration-basics). We recommend creating the `php-cgi.conf` file in Apache's `conf.d/` directory and setting these variables there. For Ubuntu systems, this is located at `/etc/apache2/conf.d/`. You may also choose to place these settings in your `/etc/apache2/httpd.conf` file. Regardless of their location, the relevant settings are:

{{< file "Apache Configuration Block" apache >}}
ScriptAlias /local-bin /usr/bin
AddHandler application/x-httpd-php5 php
Action application/x-httpd-php5 /local-bin/php-cgi

{{< /file >}}


In this example, the path to the `php-cgi` binary is `/usr/bin/php-cgi`. All files with the `php` extension will be handed to the PHP CGI binary.

You may also choose to put these configuration directives within a virtual hosting block. If you *do not* have `mod_php` enabled or installed, you can use this to selectively enable PHP for certain virtual hosts. Furthermore, if your deployment requires multiple versions of PHP, you can specify virtual host specific handlers by specifying paths to various versions of `php-cgi`.

The configuration file for the CGI executable of PHP is located at `/etc/php5/cgi/php.ini`. You can modify this file to suit the needs of your deployment.

{{< file "/etc/php5/cgi/php.ini" ini >}}
error_reporting = E_COMPILE_ERROR|E_RECOVERABLE_ERROR|E_ERROR|E_CORE_ERROR
display_errors = Off
log_errors = On
error_log = /var/log/php.log
max_execution_time = 30
memory_limit = 64M
register_globals = Off

{{< /file >}}


If you need support for MySQL in PHP, then you must install the php5-mysql package with the following command:

    apt-get install php5-mysql

When `php-cgi` is configured, you can now safely enable the `itk` message passing module for Apache. The installation process for `itk` will restart the Apache process. If you choose to use PHP CGI with the default or existing message passing module, then restart Apache by issuing the following command:

    /etc/init.d/apache2 restart

## Enabling the "itk" Message Passing Module

The default Apache configuration uses a message passing module called `worker` which uses a threaded approach to efficiently handling HTTP requests. An alternative MPM is `prefork` which does not use threads and is compatible with non-thread-safe libraries. Both the `worker` and `prefork` modules require that all requests be handled by a process running under a user with particular permissions. On Ubuntu systems, Apache processes run under the `www-data` user.

This may not be ideal if you have multiple users running publicly accessible scripts on your server. In some of these cases it is prudent to isolate virtual hosts under specific user accounts using an alternative message passing module, known as `itk` or `mpm-itk`. Functionally, `mpm-itk` is quite similar to `prefork` however `itk` can processes requests for each virtual host or each site under a specified user account. This useful in situations where you're hosting a number of distinct sites and you need to isolate sites on the basis of user privileges.

Begin by installing the mpm-itk module:

    apt-get install apache2-mpm-itk

Now, in the `<VirtualHost >` entries for your sites (the site-specific files in `/etc/apache2/sites-avalible/`) add the following sub-block:

{{< file "Apache Virtual Hosting Configuration Block" apache >}}
<IfModule mpm_itk_module>
   AssignUserId webeditor webgroup
</IfModule>

{{< /file >}}


In this example, `webeditor` is the name of the user of the specific site in question, and `webgroup` is the name of the user group that "owns" the web server related files and processes for this host. Remember that you must create the user accounts and groups using the `useradd` command. Consider our documentation of [user groups and permissions](/docs/tools-reference/linux-users-and-groups) for more information about creating the necessary users and groups.

## More Information

You may wish to consult the following resources for additional information on this topic. While these are provided in the hope that they will be useful, please note that we cannot vouch for the accuracy or timeliness of externally hosted materials.

- [The PHP Homepage](http://php.net/)
- [Apache 2.2 CGI documentation](http://httpd.apache.org/docs/2.2/howto/cgi.html)
