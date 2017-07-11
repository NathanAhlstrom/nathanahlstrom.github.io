---
layout: post
title: Elastic stack for visualizing ezProxy logs - Part 4
date: 20170711
type: post
published: true
tags: [library, ezproxy, metrics, logstash, filebeats, elasticsearch, elastic, ELK]
meta:
author: Nathan Ahlstrom
excerpt: Installing and configuring Elasticsearch of the Elastic stack in the quest to have visualized log analytics from ezProxy.
---

# Elasticsearch Setup #

This is part 4 in a series on handling ezProxy log files with filebeats, LogStash, Elasticsearch, and kibana.

The plan is to visualize ezProxy log files to see usage patterns, power users, and trends.

This how-to is for RedHat based Linux OSes running ezProxy 6.2.2 - although this should work for all versions of ezProxy.  If you have a different version of Linux, this would be a rough outline of what you need to do, but the installation steps would be different.

## Elasticsearch specifics ##

This was the hardest part of the process yet...but bear with me, we can work through it.

1) install Elasticsearch - see [previous article](/2017/06/30/ezproxy-log-visualization/){:target="_blank"} with installation instructions for rpm-based Linux systems.

2) create some directories for data and log files.

	$ sudo mkdir -p /export/home/elasticsearch/data
	$ sudo mkdir /export/home/elasticsearch/logs

These are the locations I put the data and log files, put them in a spot with enough disk space and you should be in good shape.

3) Update /etc/elasticsearch/elasticsearch.yml to know about these locations:

	# Path to directory where to store the data ...
	#
	path.data: /export/home/elasticsearch/data
	#
	# Path to log files:
	#
	path.logs: /export/home/elasticsearch/logs


4) update permissions on data / logs locations:

	$ sudo chown -R elasticsearch:elasticsearch /export/home/elasticsearch
	$ sudo chmod 4775 /export/home/elasticsearch
	$ sudo chmod 4775 /export/home/elasticsearch/data
	$ sudo chmod 4775 /export/home/elasticsearch/logs

Starting Elasticsearch as the elasticsearch user to see how it goes reveals a few updates are necessary...
	
5) Attempt to run Elasticsearch as the elasticsearch user (important):

	$ sudo su - elasticsearch
	
This switch user command might give an error, so probably necessary to update /etc/passwd to allow logging in as the elasticsearch user.  Run the following steps as yourself with sudo or root user:

	$ sudo vipw

Very careful with this file!  You can ruin your Linux box and ability to login if you make a mistake here...

Find the elasticsearch line, looks something like this one:

	elasticsearch:x:983:978:elasticsearch user:/home/elasticsearch:/sbin/nologin

Needs to be changed slightly to look like this line:

	elasticsearch:x:983:978:elasticsearch user:/export/home/elasticsearch:/bin/bash

Note: how to use vipw to change the passwd file is beyond this tutorial, be very careful.  Probably a better way to do this via [usermod](https://linux.die.net/man/8/usermod){:target="_blank"}?


Now to attempt to run elasticsearch command line as the elasticsearch user:

	$ sudo su - elasticsearch
	$ /usr/share/elasticsearch/bin/elasticsearch -v

On my system Elasticsearch gave some serious errors and warnings about the number of files it could open at one time, so the following adjustment was necessary.

6) Adjust /etc/security/limits.conf to allow elasticsearch user to open many files at once:

Add this content to /etc/security/limits.d/21-nofile.conf

	#
	# allow elasticsearch user to increase file descriptors open to 65k
	#
	elasticsearch hard nofile 65536
	elasticsearch soft nofile 65536

Log out / log back in as the elasticsearch user, you should see this when checking the setting:

	-bash-4.2$ ulimit -n
	65536

7) Final configuration elasticsearch.yml file tuning, not strictly necessary beyond what we adjusted before I believe:

	cluster.name: YourClusterNameHere

	node.name: ${HOSTNAME}

This will get you up and running on this particular localhost.

8) Finally, run Elasticsearch again:

        $ sudo su - elasticsearch
        $ /usr/share/elasticsearch/bin/elasticsearch -v

All set, now on to Kibana.
