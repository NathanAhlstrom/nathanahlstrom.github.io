---
layout: post
title: LogFormat and other logging enhancements for ezProxy - Part 1
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

### Option LogUser ###
In our case we need to see the usernames of the people using our content.  This option tells ezProxy to add the username of the person into each log file line.  For example:

	10.12.73.45 - username001 [22/Jun/2015:16:10:39 -0600] "GET https://www.examplelibraryvendor.com:443/wp-content/uploads/2015/02/su-companies_958x412-2-958x412.jpg HTTP/1.1" 200 103873
	10.12.73.45 - username001 [22/Jun/2015:16:10:39 -0600] "GET https://www.examplelibraryvendor.com:443/wp-content/uploads/2015/06/MTS_Banner_June_958x412.jpg HTTP/1.1" 200 144145
	10.12.73.45 - username001 [22/Jun/2015:16:10:40 -0600] "POST http://www.examplelibraryvendor.com:80/wp-admin/admin-ajax.php HTTP/1.1" 200 1480
	10.12.73.45 - username001 [22/Jun/2015:16:10:40 -0600] "GET http://www.examplelibraryvendor.com:80/wp-content/themes/total/images/favicon.ico HTTP/1.1" 200 1555
	10.12.73.45 - username001 [22/Jun/2015:16:10:44 -0600] "GET https://examplelibraryvendor.com:443/login/ HTTP/1.1" 301 364
	10.12.73.45 - username001 [22/Jun/2015:16:10:46 -0600] "GET https://www.examplelibraryvendor.com:443/login/ HTTP/1.1" 200 30372
	10.12.73.45 - username001 [22/Jun/2015:16:10:46 -0600] "GET https://www.examplelibraryvendor.com:443/wp-content/plugins/siteorigin-panels/css/front.css HTTP/1.1" 200 1115
	10.12.73.45 - username001 [22/Jun/2015:16:10:46 -0600] "GET https://www.examplelibraryvendor.com:443/wp-content/plugins/jquery-collapse-o-matic/light_style.css HTTP/1.1" 200 1344
	

### LogFile  ###

This line gives control over the location of the logs and the option -strftime specifies that there will be a new log file created everyday by the ezproxy software.  This is beneficial in a few ways: one, there will not be giant log files outgrowing the filesystem size limits (it has happened to me); and two, easier to set a delete pattern for logs older than retention limit.

### LogFormat ###

	LogFormat %h %l %u %t "%r" %s %b "%{referer}i" "%{user-agent}i"

Updating the LogFormat line to include the referring URL and the browser user-agent.  Possibly unnecessary, but I would like to see the browser versions for our users.  The double-quotes around the referer and user-agent are necessary.  The rest of the line is the ezproxy default.


### Glossary ###

For more information see the OCLC ezProxy documentation:

[LogFormat](https://www.oclc.org/support/services/ezproxy/documentation/cfg/logformat.en.html){:target="_blank"}

[LogFile](https://www.oclc.org/support/services/ezproxy/documentation/cfg/logfile.en.html){:target="_blank"}

[Option LogUser - see page 70 of the PDF file](https://www.oclc.org/content/dam/support/ezproxy/documentation/pdf/ezproxy_referencemanual.pdf){:target="_blank"}

[Elastic stack](https://www.elastic.co/webinars/introduction-elk-stack){:target="_blank"} - Elasticsearch, Logstash, Kibana, and Filebeats are tools for pulling log files, and pushing into a repository for the purpose of visualizing log file data (and lots of other data).

This information is current for ezproxy version 6.2.2 running on Linux.

