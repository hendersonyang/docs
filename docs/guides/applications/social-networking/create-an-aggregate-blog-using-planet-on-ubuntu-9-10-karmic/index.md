---
slug: create-an-aggregate-blog-using-planet-on-ubuntu-9-10-karmic
deprecated: true
author:
  name: Linode
  email: docs@linode.com
description: 'This guide will show you how to use The Planet feed aggregator to provide overviews of a community by collecting community feeds on Ubuntu 9.10 "Karmic".'
keywords: ["planet", "blogs", "aggregator", "feed", "rss"]
tags: ["ubuntu"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
aliases: ['/web-applications/social-networking/planet/ubuntu-9-10-karmic/','/applications/social-networking/create-an-aggregate-blog-using-planet-on-ubuntu-9-10-karmic/']
modified: 2012-10-08
modified_by:
  name: Linode
published: 2010-08-10
title: 'Create an Aggregate Blog using Planet on Ubuntu 9.10 (Karmic)'
external_resources:
    - '[The Planet''s Project''s Home Page](http://www.planetplanet.org)'
    - '[Using Cron to Schedule Tasks](/docs/linux-tools/utilities/cron)'
relations:
    platform:
        key: aggregate-blog-planet
        keywords:
            - distribution: Ubuntu 9.10
---



The Planet Feed Aggregator takes a collection of RSS feeds and generates what its founders call a "River of News" feed that combines posts from all sources into a single coherent stream. Thus, this software is useful for providing a simple and consolidated overview of ongoing output from selected blogs. Written and configured in Python and run regularly using cron, Planet is easy to configure and use.

Before beginning to follow this guide, we assume that you have completed the [Setting Up and Securing a Compute Instance](/docs/guides/set-up-and-secure/) guide. If you're new to Linux server administration, you may be interested in our [introduction to Linux concepts guide](/docs/tools-reference/introduction-to-linux-concepts/), [beginner's guide](/docs/platform/billing-and-support/linode-beginners-guide/) and [administration basics guide](/docs/tools-reference/linux-system-administration-basics/). Beyond this, Planet requires a web server to provide access to the resources it creates, but this document does not depend on specific [web server software](/docs/web-servers/) software.

## Enabling the Universe Repository

Prior to installing the Planet software, you will need to enable the `universe` repositories. Edit the `/etc/apt/sources.list` to ensure that the following lines are present:

{{< file "/etc/apt/sources.list" >}}
deb http://us.archive.ubuntu.com/ubuntu/ karmic universe
deb-src http://us.archive.ubuntu.com/ubuntu/ karmic universe

deb http://us.archive.ubuntu.com/ubuntu/ karmic-updates universe
deb-src http://us.archive.ubuntu.com/ubuntu/ karmic-updates universe

deb http://security.ubuntu.com/ubuntu karmic-security universe
deb-src http://security.ubuntu.com/ubuntu karmic-security universe

{{< /file >}}


Issue the following commands to update your system's package repositories and ensure that all installed software is up to date:

    apt-get update
    apt-get upgrade

## Install Planet Software

Install the Planet software by issuing the following command:

    apt-get install planet

This will install the Apache HTTP server as a dependency. Congratulations, we can now begin the configuration of Planet.

## Configure Planet

### Basic Planet Configuration

For the purposes of example, this document assumes that your web server is configured to use `/srv/www/example.com/public_html` as the public document root for the domain `example.com`.

Copy the default configuration file to the `/srv/www/example.com` directory:

    cp /etc/planet.conf /srv/www/example.com

Now edit the file, making sure to modify the following values to conform to the needs of your deployment. Consider the following example:

{{< file "planet.conf" >}}
name=example link=http://example.com

owner_name=username <owner_email=username@example.com>

cache_dir = /srv/www/example.com/planet_cache

output_dir = /srv/www/example.com/public_html

items_perpage = 40 days_per_page = 0

template_files = /srv/www/example.com/planet_templates/index.html.tmpl /srv/www/example.com/planet_templates/atom.xml.tmpl /srv/www/example.com/planet_templates/rss20.xml.tmpl /srv/www/example.com/planet_templates/rss10.xml.tmpl /srv/www/example.com/planet_templates/opml.xml.tmpl /srv/www/example.com/planet_templates/foafroll.xml.tmpl

[/srv/www/example.com/planet_templates/rss10.xml.tmpl] items_per_page = 30

{{< /file >}}


These settings establish the name and some background information regarding the site. The `output_dir` determines where Planet will build the site, and should point to a publicly accessible directory equivalent to or beneath the "document root" of your web server. The `items_perpage` and `days_per_page` limit the number of posts displayed on all Planet-generated pages. `items_per_page` trims the number of posts included in the feed to not surpass the threshold set. `days_per_page` sets a hard limit for the number of days of oldest possible post that can be displayed.

The remaining settings control the behavior and use of templates. Issue the following commands to copy the default templates and other support files into a site specific configuration:

    mkdir -p /srv/www/example.com/planet_cache/
    mkdir -p /srv/www/example.com/planet_templates/
    cp /var/lib/planet/templates/* /srv/www/example.com/planet_templates/
    cp -R /var/lib/planet/www/images/ /srv/www/example.com/public_html/
    cp /usr/share/doc/planet/examples/planet.* /srv/www/example.com/public_html/

If you want to override any of the default values like the encoding or the value of `items_per_page` as seen above, simply create an item block beginning with the full path to the template and specify the values beneath. If you need planet to generate an additional template, simply add the full path to the template to the end of the `template_files` list.

### Configuring Aggregation

At the end of your `planet.conf` file, add entries that resemble the following for each feed that you would like to collect in the Planet you're building:

{{< file "planet.conf" >}}
[<https://www.linode.com/docs/rss>]
name = Linode

{{< /file >}}

If you want to take advantage of Planet's support for per-feed "faces" or avatars to identify each feed with a distinct logo or image, consider the following examples:

{{< file "planet.conf" >}}
[DEFAULT] facewidth = 64 faceheight = 64

[<https://www.linode.com/docs/rss>] name = Linode face = lin-lib-avatar.png

{{< /file >}}


You can specify default "width" and "height" in the `[DEFAULT]` directive, but these values can be overridden for feed-specific settings. All "faces" should be stored in an `images/` directory beneath the output directory. In the case of this example, deposit images into `/srv/www/example.com/public_html/images/`.

Once you have completed your modifications to `planet.conf`, run Planet for the first time by issuing the following command:

    planetplanet /srv/www/example.com/planet.conf

The Planet software will only poll the source feeds when the above command is used. Otherwise, all files generated by Planet are static. By maintaining multiple `planet.conf` files and specifying distinct output directories, it's possible to generate multiple Planet-based websites on a single server.

## Running Planet

While you can run Planet without incident using the above method, we recommend running planet regularly using a "cronjob." Edit your crontab file with the following command:

    crontab -e

Insert the following job into the crontab:

{{< file "crontab" >}}
*/10* * * * planetplanet /srv/www/example.com/planet.conf

{{< /file >}}


Save the crontab. Your Planet generated site will refresh every 10 minutes. Congratulations!
