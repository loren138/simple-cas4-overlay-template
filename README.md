MySQL, OAuth, and Google Apps CAS4 Overlay Template
============================

IMPORTANT STEP 1: Copy cas.properties.template to cas.properties!  (This ideally prevents your credentials from being checked into git.)

I've tried to put good comments around things so you can look at the file changes and remove features you don't need.

We never actually used the Google Apps SSO part of this template because that doesn't add SSO to Gmail for IOS so we decided that it wasn't worth it since we'd still need to sync passwords with gmail.  We tried it once with CAS 3, and it failed and locked up our account for a day due to propogation while turning it off so if you use it, be sure to set the IP filter to just your machine first to see if it will work before enabling it for everyone.

I also had a difficult time getting tomcat7 running with SSL so if you need help with that create an issue, and I'll try to add a page on that.

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

# Google Apps SAML Integration
http://jasig.github.io/cas/4.0.x/protocol/SAML-Protocol.html

Generate these keys into /etc/cas/

```
openssl genrsa -out private.key 1024
openssl rsa -pubout -in private.key -out public.key -inform PEM -outform DER
openssl pkcs8 -topk8 -inform PER -outform DER -nocrypt -in private.key -out private.p8
openssl req -new -x509 -key private.key -out x509.pem -days 365

sudo chown tomcat7:tomcat7 private.key private.p8 
```

## Configuring Google Apps
Use the following URLs when you are configuring for Google Apps

Sign-in page URL: https://yourCasServer/login
Sign-out page URL: https://yourCasServer/logout
Change password URL: http://whateverServerYouWouldLike

# CAS OAuth2 Support
This adds the web.xml and cas-servlet.xml files.  If you don't want OAuth support, remove both files from `src/main/webapp/WEB-INF` and the dependency from `pom.xml`.  
If you do need CAS support configure, `loginUrl` and `timeout` in `cas-servlet.xml`.
See: http://jasig.github.io/cas/4.0.x/installation/OAuth-OpenId-Authentication.html

For how to use the OAuth support, see http://jasig.github.io/cas/4.0.x/protocol/OAuth-Protocol.html.

You will need to configure your services in `/src/main/webapp/WEB-INF/deployerConfigContext.xml`.

# Start of original Readme
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
