MsSQL, MySQL, OAuth, and Google Apps CAS4 Overlay Template
============================
For tomcat setup see tomcatSetup.md (also in this repo).

If you don't want MSSQL or to deploy this at root, you back a few commits in the repo.  If you want to help organize the readme, feel free to do so and send a pull request.

IMPORTANT STEP 1: Copy cas.properties.template to cas.properties!  (This ideally prevents your credentials from being checked into git.)

We never actually used the Google Apps SSO part of this template because that doesn't add SSO to Gmail for IOS so we decided that it wasn't worth it since we'd still need to sync passwords with gmail.  We tried it once with CAS 3, and it failed and locked up our account for a day due to propogation while turning it off so if you use it, be sure to set the IP filter to just your machine first to see if it will work before enabling it for everyone.

Before running this, you will want to have at least installed the following on Ubuntu:

`sudo apt-get install tomcat7 maven ant maven-ant-helper openjdk-7-jdk libmysql-java`

I've now made this deploy at root on the web server.  If you want it in the cas directory on the server, edit build.xml and change
the project name from ROOT to cas. (`<project name="ROOT" default="deploy" basedir=".">`)

If you want to keep it at root, you'll need to `sudo rm -rf /var/lib/tomcat7/webapps/ROOT`
I've tried to put good comments around things so you can look at the file changes and remove features you don't need.

For Ubuntu 14.04 in your .bashrc add the following lines:

    export CATALINA_HOME=/usr/share/tomcat7
    export MAVEN_HOME=/usr/share/maven
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/jre


You may also need to add these sym links:

     cd /usr/share/tomcat
     ln -s /var/lib/tomcat7/webapps/ ./webapps
     ln -s /var/lib/tomcat7/common/ ./common
     sudo ln -s /var/lib/tomcat7/server/ ./server
     sudo ln -s /var/lib/tomcat7/shared/ ./shared
     
     cd /usr/share/tomcat7/lib
     sudo ln -s ../../java/mysql-connector-java.jar mysql-connector-java.jar

To Setup the CAS log file at the default location:
```
sudo mkdir /var/log/cas
sudo chown tomcat7:tomcat7 /var/log/cas
sudo touch /var/log/cas/cas.log
sudo chown tomcat7:tomcat7 /var/log/cas/cas.log
```

Note: Error messages may be found in the cas log (/var/log/cas/cas.log) but more expansive ones are likely in the tomcat7 log at (/var/log/tomcat7/catalina.log).

#Building Cas

I use this alais `alias casBuild='sudo rm -rf /var/lib/tomcat7/webapps/ROOT/ && sudo service tomcat7 stop && ant deploy && sudo service tomcat7 start'`

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

# SQL Server Support

* Get the SQL Server JAR from https://www.microsoft.com/en-us/download/details.aspx?displaylang=en&id=11774 (you may need to use IE to get the download to actually start)
* Extract it `tar -xvf sqljdbc_4.2.6420.100_enu.tar.gz ` and `sudo mv ./sqljdbc_4.2/enu/sqljdbc41.jar /usr/share/java/`
* `sudo chown root:root /usr/share/java/sqljdbc41.jar`
* `cd /usr/share/tomcat7/lib` && `sudo ln -s ../../java/sqljdbc41.jar ./`

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
