---
layout: post
title: Kibana for visualizing ezProxy logs - Part 5
date: 20170714
type: post
published: true
tags: [library, ezproxy, metrics, logstash, filebeats, elasticsearch, elastic, ELK]
meta:
author: Nathan Ahlstrom
excerpt: Installing and configuring Kibana of the Elastic stack in the quest to have visualized log analytics from ezProxy.
---

# So far in this series... #

This is part 5 in a series on handling ezProxy log files with filebeats, LogStash, Elasticsearch, and Kibana.

The plan is to visualize ezProxy log files to see usage patterns, power users, and trends.

This how-to is for RedHat based Linux OSes running ezProxy 6.2.2 - although this should work for all versions of ezProxy.  If you have a different version of Linux, this would be a rough outline of what you need to do, but the installation steps would be different.

# Kibana Setup #

After the last installment about installing Elasticsearch (the hardest part of the process), here we are at Kibana, which is the easiest part of the process.

1) install Kibana - see [previous article](/2017/06/30/ezproxy-log-visualization/){:target="_blank"} with installation instructions for rpm-based Linux systems.

2) Modify the YML file: /etc/kibana/kibana.yml

	# To allow connections from remote users, set this parameter to a non-loopback address.
	#server.host: "localhost"
	server.host: "logvis.example.com"

3) Start Kibana:

	$ sudo /usr/share/kibana/bin/kibana &

The ampersand at the end of the command line tells Kibana to run in the background...good for the moment.  You should see a bit of output ending with something like this:

	  [21:36:07.851] [info][status][ui settings] Status changed from uninitialized to green - Ready

4) Point your browser at http://logvis.example.com:5601/

If everything went as planned your browser should fire up Kibana and you will begin to see logstash-* data from your ezProxy log files here.

There is a lot more to adjust: 

1.  Tune LogStash filters to set the bytes as a number field 
2.  Set timestamp column as date
3.  Set LogStash, Elasticsearch, and Kibana as Linux services.
4.  Learn how to use Kibana!

Enjoy!



