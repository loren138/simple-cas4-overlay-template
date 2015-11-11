Installing Tomcat with SSL on Port 80/443 on Ubuntu 14.04
=========================================================

This tutorial is somewhat useful for getting tomcat up and running but not necessary.  I'd skip reading it.
https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-7-on-ubuntu-14-04-via-apt-get

You should update to tomcat8 on future ubuntu versions, but I have no idea how that will impact this process.

     sudo apt-get update
     # One doc mentions maven2, you don't want that, you want maven (which installs maven3...)
     sudo apt-get install tomcat7 maven ant maven-ant-helper openjdk-7-jdk libmysql-java libtcnative-1 apache2
     # I didn't use these packages (mentioned in some of the docs)
     #sudo apt-get install tomcat7-docs tomcat7-admin

For ease of use, you should add yourself to the tomcat7 group.

This tutorial is quite useful, but you have to change a couple of things since it's written for ubuntu 12.04 and cas 3.5.2.
Mainly just use it to get instructions for installing SSL into tomcat.  Do not use the apt-get install line from the docs.
Thus, you should stop with this tutorial when you get to, "Setting Up The CAS WAR Overlay".  A couple changes/additions to this
doc is below.

http://kogentadono.com/2014/10/16/installing-cas-3-5-2-on-ubuntu-12-04-part-1-tomcat-7-and-cas/

For better SSL protocol support, we'll set up a proxy through apache to tomcat.

In `/etc/tomcat7/server.xml`:  Comment out the default connector and add on AJP connector.
```
    <!--
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="UTF-8"
               redirectPort="8443" />
    -->
    <Connector port="8009" protocol="AJP/1.3" maxConnections="256" keepAliveTimeout="30000" redirectPort="8443" />
```

In `/etc/defaults/tomcat7`, I merged the JAVA_OPTS between the two docs so my line is

	JAVA_OPTS="-Djava.awt.headless=true -Xms512M -Xmx1024M -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=512m -XX:+CMSIncrementalMode"

	sudo rm /etc/authbind/byport/80
	sudo rm /etc/authbind/byport/443
	Remove authbind=yes (set to no)
	
	You need to comment
`<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />` in the `/etc/tomcat7/server.xml` file.

#Setting Up Apache

You should edit `/var/www/html/index.html` to something for default hits to the server.

```sudo a2enmod proxy && sudo a2enmod proxy_http && sudo a2enmod ssl && sudo a2enmod proxy_ajp && sudo a2enmod rewrite```

In your Apache /etc/apache/mods-available/ssl.conf file replace
```
        #SSLCipherSuite RC4-SHA:AES128-SHA:HIGH:MEDIUM:!aNULL:!MD5
        #SSLHonorCipherOrder on

        #   The protocols to enable.
        #   Available values: all, SSLv3, TLSv1, TLSv1.1, TLSv1.2
        #   SSL v2  is no longer supported
        SSLProtocol all
```
with
```
        SSLHonorCipherOrder on
        SSLCipherSuite "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA RC4 !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS"

        #   The protocols to enable.
        #   Available values: all, SSLv3, TLSv1, TLSv1.1, TLSv1.2
        #   SSL v2  is no longer supported
        SSLProtocol all -SSLv3
```

Set up your config files:
If you are running cas as the default option, you will want to disable the default site or put these items in the 000-default config files.

/etc/apache2/sites-available/cas.conf
```
<VirtualHost *:80>
    # This will enable the Rewrite capabilities
    RewriteEngine On

    # This checks to make sure the connection is not already HTTPS
    RewriteCond %{HTTPS} !=on

    # This rule will redirect users from their original location, to the same location but using HTTPS.
    # i.e.  http://www.example.com/foo/ to https://www.example.com/foo/
    # The leading slash is made optional so that this will work either in httpd.conf
    # or .htaccess context
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R,L]

    #ProxyPreserveHost On
    ProxyPass "/" "ajp://localhost:8009/"
    ProxyPassReverse "/" "ajp://localhost:8009/"
    ServerName webdev-f.sbts.edu
</VirtualHost>
<VirtualHost *:443>
    #ProxyPreserveHost On

    <IfModule env_module>
         # Fake SSL if Loadbalancer does SSL-Offload 
         SetEnvIf Front-End-Https "^on$" HTTPS=on
    </IfModule>

    SSLEngine on
    SSLCertificateFile "/etc/apache2/ssl.crt/file.crt"
    SSLCertificateKeyFile "/etc/apache2/ssl.key/file.key"
    SSLCertificateChainFile "/etc/apache2/ssl.crt/gd_bundle-g2-g1-issued-20140411.crt"

    ProxyPass "/" "ajp://localhost:8009/"
    ProxyPassReverse "/" "ajp://localhost:8009/"
    ServerName webdev-f.sbts.edu:443
</VirtualHost>
```

```sudo a2ensite cas.conf && sudo service apache2 reload```

# Install CAS

See README.md

