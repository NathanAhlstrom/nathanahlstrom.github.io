---
layout: post
title: Elastic stack for visualizing ezProxy logs - Part 2 
date: 20170707
type: post
published: true
tags: [library, ezproxy, metrics, logstash, filebeats, elasticsearch, elastic, ELK]
meta:
author: Nathan Ahlstrom
excerpt: Installing and configuring LogStash of the Elastic stack to visualize ezProxy logs.
---

# LogStash from Elastic for processing log file streams #

This is part 3 in a series on handling ezProxy log files with filebeats, LogStash, elasticsearch, and kibana.

The plan is to visualize ezProxy log files to see usage patterns, power users, and trends.

This how-to is for RedHat based Linux OSes running ezProxy 6.2.2 - although this should work for all versions of ezProxy.  If you have a different version of Linux, this would be a rough outline of what you need to do, but the installation steps would be different.

## LogStash specifics ##

LogStash has roughly three main sections of the configuration file.  Input, Output, and filter.

This is my basic example modeled from the [LogStash documentation](https://www.elastic.co/guide/en/logstash/current/advanced-pipeline.html#configuring-file-input){:target="_blank"}:

	input {
		beats { 
			port => "5043"
		}
	}
	
	filter {
		grok {
			match => { "message" => "%{HTTPD_COMMONLOG}" }
		}
	}
	
	output {
		elasticsearch { hosts => ["localhost:9200"] }
	}
	
Save this as first-pipeline.conf (or whatever .conf name you would like) and put it into /etc/logstash.

Let's take a look at each section.

### Input ###

This part of the configuration file is instruction for LogStash to look for input (ezProxy log activity) from filebeats talking to LogStash on port 5043.  LogStash will listen on port 5043, beats should be speaking to this port (might have to tweaks the filebeats.yml file).

### Filter ###

This is a bit more confusing section.  This is where LogStash parses the input and you can tell it what format of data to be expecting and where to put it.  So in this case [HTTPD_COMMONLOG](https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/httpd){:target="_blank"} is a standard format that LogStash knows about and the usual format for Apache and other HTTPD servers logs.  If you adjusted your ezProxy LogFormat line per my [previous post](/2017/06/29/ezproxy-logging-config/){:target="_blank"}, ezProxy will log in this same format.

LogStash filters every line in the log file and puts the data into specific named fields.

### Output ###

The output line tells LogStash where to send the data next - in this case the elasticsearch service for storage (essentially).

Setting up elasticsearch was the most difficult part of this adventure, we'll get to that in a soon to come post here.

### Running LogStash ###

So we have filebeats running and publishing log data...how do we turn on LogStash to listen?

First to run a configuration test with the "-t" option:

	$ sudo /usr/share/logstash/bin/logstash --path.settings /etc/logstash -f /etc/logstash/first-pipeline.conf -t

Now to actually run the LogStash service with the "-r" option and the "&" to put it into the background:

	$ sudo /usr/share/logstash/bin/logstash --path.settings /etc/logstash -f /etc/logstash/first-pipeline.conf -r &

It should startup and do exactly nothing visible, but silently be consuming log files and attempting to send to elasticsearch.

Until next time.
