---
slug: installing-prosody-xmpp-server-on-ubuntu-9-04-jaunty
deprecated: true
author:
  name: Linode
  email: docs@linode.com
description: 'Installation and basic usage guide for Prosody, a lightweight XMPP server on Ubuntu 9.04 (Jaunty).'
keywords: ["prosody", "prosody ubuntu jaunty", "prosody.im", "xmpp", "real time messaging", "lua"]
tags: ["ubuntu"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
aliases: ['/communications/xmpp/prosody/ubuntu-9-04-jaunty/','/applications/messaging/installing-prosody-xmpp-server-on-ubuntu-9-04-jaunty/']
modified: 2011-04-29
modified_by:
  name: Linode
published: 2009-10-13
title: 'Installing Prosody XMPP Server on Ubuntu 9.04 (Jaunty)'
relations:
    platform:
        key: how-to-install-prosody
        keywords:
            - distribution: Ubuntu 9.04
---



Prosody is a XMPP/Jabber server programmed in Lua that is simple and lightweight. Prosody uses fewer resources than its counterparts and is designed to be easy to configure and run. [ejabberd](/docs/applications/messaging/use-ejabberd-for-instant-messaging-on-ubuntu-12-04/) or [OpenFire](/docs/applications/messaging/install-openfire-on-ubuntu-12-04-for-instant-messaging/) may be better suited for larger applications, but for most independent and small scale uses Prosody is a more resource-efficient solution. Prosody is a very good candidate for running an XMPP server for a very small base of users, or for XMPP development.

Before we begin with the installation and configuration of Prosody, we assume that you have a running and up to date installation of Ubuntu Jaunty, have completed our [Setting Up and Securing a Compute Instance](/docs/guides/set-up-and-secure/) guide and have logged in via SSH as root.

## Adding Software Repositories

The developers of Prosody provide software repositories for Debian and Ubuntu to more effectively distribute current versions of the software to users. In order to make these repositories accessible to your system we must append the following line to the `/etc/apt/sources.list` file. For Jaunty, this will also include enabling the `universe` repository. Edit your `/etc/apt/sources.list` file to resemble this example by removing the hash symbol in front of the `universe` lines:

{{< file "/etc/apt/sources.list" >}}
## main & restricted repositories
deb http://us.archive.ubuntu.com/ubuntu/ jaunty main restricted
deb-src http://us.archive.ubuntu.com/ubuntu/ jaunty main restricted

deb http://security.ubuntu.com/ubuntu jaunty-security main restricted
deb-src http://security.ubuntu.com/ubuntu jaunty-security main restricted

## universe repositories
deb http://us.archive.ubuntu.com/ubuntu/ jaunty universe
deb-src http://us.archive.ubuntu.com/ubuntu/ jaunty universe
deb http://us.archive.ubuntu.com/ubuntu/ jaunty-updates universe
deb-src http://us.archive.ubuntu.com/ubuntu/ jaunty-updates universe

deb http://security.ubuntu.com/ubuntu jaunty-security universe
deb-src http://security.ubuntu.com/ubuntu jaunty-security universe

{{< /file >}}


At the end of the file, also insert this line for the Prosody repository:

{{< file "/etc/apt/sources.list" >}}
deb http://packages.prosody.im/debian jaunty main

{{< /file >}}


Now, to download the public key for the Prosody package repository, issue the following `wget` command. (You may need to install `wget` first by running `apt-get install wget`). This will allow you to authenticate and verify packages:

    wget http://prosody.im/files/prosody-debian-packages.key -O- | apt-key add -

You can now issue the following command to refresh the package database:

    apt-get update
    apt-get upgrade

## Install Prosody

With the proper repository enabled, we're now ready to install the Prosody server. Use the following command:

    apt-get install prosody liblua5.1-sec0

When `apt` finishes, the Prosody server will have been successfully installed, and will be ready for configuration. Prosody provides an init script that allows you to reload the configuration file, start, stop, or restart the XMPP server. Issue one of the following commands as appropriate:

    /etc/init.d/prosody reload
    /etc/init.d/prosody start
    /etc/init.d/prosody stop
    /etc/init.d/prosody restart

## Configure Prosody Server

The configuration file for Prosody is located in `/etc/prosody/prosody.cfg.lua`, and is written in Lua syntax.

Note that in the Lua programing language, comments (lines that are ignored by the interpreter) are preceded by two hyphen characters (e.g. `--`). The default config has some basic instructions in Lua syntax, which can be helpful if you're unfamiliar with the language.

To allow Prosody to provide XMPP/jabber services for more than one domain, insert a line in the following form into the configuration file. This example defines three virtual hosts.

{{< file "/etc/prosody/prosody.cfg.lua" lua >}}
Host "example.com"
Host "example.com"
Host "staff.example.com"

{{< /file >}}


Following a `Host` line there are generally a series of host-specific configuration options. If you want to set options for all hosts, add them below the `Host "*"` entry in your config file. For instance, to ensure that Prosody behaves like a proper Linux server daemon make sure that the `posix;` option is included in the `modules_enabled = { }` table.

{{< file "/etc/prosody/prosody.cfg.lua" lua >}}
modules_enabled = {
                  -- [...]
                  "posix";
                  -- [...]
                  }

{{< /file >}}


Note that there should be a number of global modules included in this table to provide basic functionality.

To disable a host without removing it from your configuration file, add the following line to its section of the file:

{{< file "/etc/prosody/prosody.cfg.lua" lua >}}
enabled = false

{{< /file >}}


To specify administrators for your server, add a line in the following format to your `prosody.cfg.lua` file.

{{< file "/etc/prosody/prosody.cfg.lua" lua >}}
admins = { "admin1@example.com", "admin2@example.com" }

{{< /file >}}


To add server-wide administrators, add the `admins` line to the `Hosts "*"` section. To grant specific users more granular control to administer particular hosts, you can add an `admins` line, or more properly tables in Lua, to specific hosts.

If you need to enable the legacy SSL/TLS support, add the following line specifying the port on which the server should listen for these connections.

{{< file "/etc/prosody/prosody.cfg.lua" lua >}}
legacy_ssl_ports = { 5223 }

{{< /file >}}


Do not forget to reload the configuration for the Prosody server after making any changes to your `/etc/prosody/prosody.cfg.lua` file, by issuing the following command:

    /etc/init.d/prosody reload

## XMPP Federation and DNS

To ensure that your Prosody instance will federate properly with the rest of the XMPP network, particularly with Google's "GTalk" service (i.e. the ["@gmail.com](mailto:"@gmail.com)" chat tool,) we must set the SRV records for the domain to point to the server where the Prosody instance is running. We need three records, which can be created in the DNS Management tool of your choice:

1.  Service: `_xmpp-server` Protocol: TCP Port: 5269
2.  Service: `_xmpp-client` Protocol: TCP Port: 5222
3.  Service: `_jabber` Protocol: TCP Port: 5269

The "target" of the SRV record should point to the publicly routable hostname for that machine (e.g. "username.example.com"). The priority and weight should both be set to `0`.

## Enabling Components

In the XMPP world, many services are provided in components, which allows for greater ease of customization within a basic framework. A common example of this is the MUC or multi-user chat functionality. To enable MUC services in Prosody you need to add a line like the following to your `/etc/prosody/prosody.cfg.lua` file.

{{< file "/etc/prosody/prosody.cfg.lua" lua >}}
Component "conference.example.com" "muc"

{{< /file >}}


In this example, `conference.example.com` is the domain where the MUC rooms are located, and will require an "[DNS A record,](/docs/networking/dns/dns-records-an-introduction/)" that points to the IP Address where the Prosody instance is running. MUCs will be identified as JIDs (Jabber IDs) at this hostname, so for instance the "rabbits" MUC hosted by this server would be located at `rabbits@conference.example.com`.

MUC, in contrast to many other common components in the XMPP world, is provided internally by Prosody. Other components, like transports to other services, run on an external interface. Each external component has its own host name, and provides a secret key which allows the central server to authenticate to it. See the following "aim.example.com" component as an example.

{{< file "/etc/prosody/prosody.cfg.lua" lua >}}
Component "aim.example.com"
component_secret = "mysecretcomponentpassword"

{{< /file >}}


Note that external components will need to be installed and configured independently of Prosody.

Typically, Prosody listens for connections from components on the localhost interface (i.e. on the `127.0.0.1` interface;). If you're connected to external resources that are running on an alternate interface, specify the following variables as appropriate in the `Host "*"` section of the file config file.

{{< file "/etc/prosody/prosody.cfg.lua" lua >}}
Host "*"

     component_interface = "192.168.0.10"
     component_ports = { 8888, 8887 }

{{< /file >}}


## Using prosodyctl

The XMPP protocol supports "in-band" registration, where users can register for accounts with your server via the XMPP interface. However, this is often an undesirable function as it doesn't permit the server administrator the ability to moderate the creation of new accounts and can lead to spam-related problems. As a result, Prosody has this functionality disabled by default. While you can enable in-band registration, we recommend using the `prosodyctl` interface at the terminal prompt.

If you're familiar with the `ejabberdctl` interface from [ejabberd,](/docs/applications/messaging/use-ejabberd-for-instant-messaging-on-ubuntu-12-04/) `prosodyctl` mimics its counterpart as much as possible.

To use `prosodyctl` to register a user, in this case `lollipop@example.com`, issue the following command:

    prosodyctl adduser lollipop@example.com

To set the password for this account, issue the following command and enter the password as requested:

    prosodyctl passwd lollipop@example.com

To remove this user, issue the following command:

    prosodyctl deluser lollipop@example.com

Additionally, `prosodyctl` can provide a report on the status of the server in response to the following command:

    prosodyctl status

Note that all of the `prosodyctl` commands require root privileges, unless you've logged in as the same user that Prosody runs under (not recommended). Congratulations, Prosody is now installed and configured, and you may begin using it to power your real time communications needs.

## More Information

You may wish to consult the following resources for additional information on this topic. While these are provided in the hope that they will be useful, please note that we cannot vouch for the accuracy or timeliness of externally hosted materials.

- [The official Prosody server website](http://prosody.im/)
- [XMPP Standards Foundation](http://xmpp.org/)
- [XMPP Client Software](http://xmpp.org/software/clients.shtml)



