---
layout: post
title: LogFormat and other logging enhancements for ezProxy
date: 20170629
type: post
published: true
tags: [library, ezproxy, metrics, logstash, filebeats, elasticsearch, elastic, ELK]
meta:
author: Nathan Ahlstrom
excerpt: Tuning up logging on ezproxy in preparation to visualize using the Elastic stack.
---
# LogFormat, LogFile, and other tune-ups to ezProxy in preparation for visualization of library database usage. #

I am working on getting our ezproxy prepared for rolling out the Elastic stack to visualize the ezProxy log files.

A few changes in our config.txt to be prepared:

	Option	LogUser
	LogFile	-strftime /opt/ezproxy/logs/ezproxy-%Y%m%d.log
	LogFormat %h %l %u %t "%r" %s %b "%{referer}i" "%{user-agent}i"

This also requires creating the /opt/ezproxy/logs directory.

In our case we need to see the usernames of the people using our content.  

Updating the LogFormat line to include the referring URL and the browser user-agent.

For more information see the OCLC ezProxy documentation:

[LogFormat](https://www.oclc.org/support/services/ezproxy/documentation/cfg/logformat.en.html){:target="_blank"}

[LogFile](https://www.oclc.org/support/services/ezproxy/documentation/cfg/logfile.en.html){:target="_blank"}

[Option LogUser - see page 70 of the PDF file](https://www.oclc.org/content/dam/support/ezproxy/documentation/pdf/ezproxy_referencemanual.pdf){:target="_blank"}

This information is current for ezproxy version 6.2.2 running on Linux.

[Elastic stack](https://www.elastic.co/webinars/introduction-elk-stack){:target="_blank"} - Elasticsearch, Logstash, Kibana, and Filebeats are tools for pulling log files, and pushing into a repository for the purpose of visualizing log file data (and lots of other data).


