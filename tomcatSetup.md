Installing Tomcat with SSL on Port 80/443 on Ubuntu 14.04
=========================================================

This tutorial is somewhat useful for getting tomcat up and running but not necessary.  I'd skip reading it.
https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-7-on-ubuntu-14-04-via-apt-get

You should update to tomcat8 on future ubuntu versions, but I have no idea how that will impact this process.

     sudo apt-get update
     # One doc mentions maven2, you don't want that, you want maven (which installs maven3...)
     sudo apt-get install tomcat7 maven ant maven-ant-helper openjdk-7-jdk mysql libmysql-java libtcnative-1
     # I didn't use these packages (mentioned in some of the docs)
     #sudo apt-get install tomcat7-docs tomcat7-admin

For ease of use, you should add yourself to the tomcat7 group.

This tutorial is quite useful, but you have to change a couple of things since it's written for ubuntu 12.04 and cas 3.5.2.
Mainly just use it to get instructions for installing SSL into tomcat.  Do not use the apt-get install line from the docs.
Thus, you should stop with this tutorial when you get to, "Setting Up The CAS WAR Overlay".  A couple changes/additions to this
doc is below.

http://kogentadono.com/2014/10/16/installing-cas-3-5-2-on-ubuntu-12-04-part-1-tomcat-7-and-cas/

In `/etc/defaults/tomcat7`, I merged the JAVA_OPTS between the two docs so my line is

	JAVA_OPTS="-Djava.security.egd=file:/dev/./urandom -Djava.awt.headless=true -Xms512M -Xmx1024M -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=512m -XX:+CMSIncrementalMode -Dhttps.protocols=TLSv1"


You need to uncomment
`<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />` in the `/etc/tomcat7/server.xml` file.

If you change tomcat to port 80/443, you may need to run the following code: 
(http://stackoverflow.com/questions/23272666/tomcat7-bind-to-port-80-fails-in-ubuntu-14-04lts)

	sudo touch /etc/authbind/byport/80
	sudo chmod 500 /etc/authbind/byport/80
	sudo chown tomcat7 /etc/authbind/byport/80
	sudo touch /etc/authbind/byport/443
	sudo chmod 500 /etc/authbind/byport/443
	sudo chown tomcat7 /etc/authbind/byport/443


# Install CAS

See README.md

