MySQL CAS4 Overlay Template
============================

IMPORTANT STEP 1: Copy cas.properties.template to cas.properties!  (This ideally prevents your credentials from being checked into git.)

Note: I'm aiming for 1 commit per feature I add so if you need only MySQL or OAuth2 or Attributes, you can use commits and the diffs to discover what has been done.  I also had a difficult time getting tomcat7 running with SSL so if you need help with that create an issue, and I'll try to add a page on that.

Before running this, you will want to have at least installed the following on Ubuntu:

`sudo apt-get install tomcat7 maven ant maven-ant-helper openjdk-7-jdk mysql libmysql-java`

Again for Ubuntu 14.04 in your .bashrc add the following lines:

```
export CATALINA_HOME=/usr/share/tomcat7
export MAVEN_HOME=/usr/share/maven
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/jre```


You may also need to add these sym links:

```
cd /usr/share/tomcat
ln -s /var/lib/tomcat7/webapps/ ./webapps
ln -s /var/lib/tomcat7/common/ ./common
sudo ln -s /var/lib/tomcat7/server/ ./server
sudo ln -s /var/lib/tomcat7/shared/ ./shared

cd /usr/share/tomcat7/lib
sudo ln -s ../../java/mysql-connector-java.jar mysql-connector-java.jar
```

To Setup the CAS log file at the default location:
```
sudo mkdir /var/log/cas
sudo chown tomcat7:tomcat7 /var/log/cas
sudo touch /var/log/cas/cas.log
sudo chown tomcat7:tomcat7 /var/log/cas/cas.log
```

Note: Error messages may be found in the cas log (/var/log/cas/cas.log) but more expansive ones are likely in the tomcat7 log at (/var/log/tomcat7/catalina.log).

Generic CAS maven war overlay to exercise the latest versions of CAS 4.x line. This overlay could be freely used as a starting template for local CAS maven war overlays.

# Versions
```xml
<cas.version>4.0.4</cas.version>
```

# Recommended Requirements
* JDK 1.7+
* Apache Maven 3+
* Servlet container supporting Servlet 3+ spec (e.g. Apache Tomcat 7+)

# Configuration
The `etc` directory contains the sample configuration files that would need to be copied to an external file system location (`/etc/cas` by default)
and configured to satisfy local CAS installation needs. Current files are:

* `cas.properties`
* `log4j.xml`

# Deployment

## Maven
* Execute `mvn clean package`
* Deploy resultant `target/cas.war` to a Servlet container of choice.

## Ant

* Define `CATALINA_HOME` and `MAVEN_HOME`
* Execute `ant deploy`
* For Ubuntu, I use `sudo rm -rf /var/lib/tomcat7/webapps/cas/ && sudo service tomcat7 stop && ant deploy && sudo service tomcat7 start`
