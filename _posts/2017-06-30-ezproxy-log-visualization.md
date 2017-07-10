---
layout: post
title: Elastic stack for visualizing ezproxy log data - Part 2
date: 20170630
type: post
published: true
tags: [library, ezproxy, metrics, logstash, filebeats, elasticsearch, elastic, ELK]
meta:
author: Nathan Ahlstrom
excerpt: Installing and configuring the Elastic stack to visualize ezproxy log files - Part 2 of series
---

# Amazing visuals of ezProxy log files #

Attempting to visualize our ezproxy log files to see usage patterns, power users, and trends.  Through the ezproxy listserv heard about someone using logstash and kibana to do this, although I cannot find that reference right now.

This how-to is for RedHat based Linux OSes running ezproxy 6.2.2 - although this should work for all versions of ezProxy.  If you have a different version of Linux, this would be a rough outline of what you need to do, but the installation steps would be different.

Moving right along to the nuts and bolts:

__1) Find__ the correct packages from [https://www.elastic.co/downloads](https://www.elastic.co/downloads){:target="_blank"}, for this how-to, we will need: elasticsearch, kibana, beats, and logstash.


__2) Download__ the latest RPM files from [above URL](https://www.elastic.co/downloads){:target="_blank"}, based on the hardware platform from as told by uname -i:

	mkdir rpms
	cd rpms
	wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.3.rpm
	wget https://artifacts.elastic.co/downloads/kibana/kibana-5.4.3-x86_64.rpm
	wget https://artifacts.elastic.co/downloads/logstash/logstash-5.4.3.rpm
	wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.4.3-x86_64.rpm


__3) Install__ the RPM packages:

	sudo yum -y install *.rpm

The -y on the installation says to answer Yes to the are you sure prompt.  You can press "y" on your own if you like.

The sudo command allows you to run commands with extra privileges, if you do not have sudo rights, you will need to speak to your system administrator.


__4) Setup__ filebeats

The filebeats configuration file at /etc/filebeat/filebeat.yml needs to have this line:

	#=========================== Filebeat prospectors =============================
	
	filebeat.prospectors:
	
	# Each - is a prospector. Most options can be set at the prospector level, so
	# you can use different prospectors for various configurations.
	# Below are the prospector specific configurations.
	
	- input_type: log
	
	  # Paths that should be crawled and fetched. Glob based paths.
	  paths:
	    #- /var/log/*.log
	    #- c:\programdata\elasticsearch\logs\*
	     - /opt/ezproxy/logs/*
	

The default RHEL installation of filebeats put the configuration files into /etc/filebeat.  Edit the file filebeat.yml in that config directory with your favorite editor.  The "input_type: log" section should have a line pointing to the ezproxy log files, in my example "- /opt/ezproxy/logs/*" all the other input log lines are commented out with a "#".

In the output section, I have commented out the elasticsearch text entirely...strange, were we not going to use elasticsearch in the project? Yes, but later on.

	#-------------------------- Elasticsearch output ------------------------------
	#output.elasticsearch:
	  # Array of hosts to connect to.
	  #  hosts: ["localhost:9200"]

Enable by removing the "#"/comment symbol on the output.logstash line.

	#----------------------------- Logstash output --------------------------------
	output.logstash:
  	  # The Logstash hosts
  	  hosts: ["localhost:5044"]

In this section we are pointing to a locally running logstash service, which we have not yet configured.  Before we get to that point, let's run the filebeat program to test it out.

	$ sudo /usr/bin/filebeat.sh

This command requires elevation via sudo to have permissions to read the log files and the filebeat configuration.  It doesn't say much since it can't push the logs anywhere yet.  So next step is to setup logstash.

This post is getting log and wordy, so I'm planning to tackle logstash in the next installment.

Hope this is helpful.


