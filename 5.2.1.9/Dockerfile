FROM tomcat:8.0.43-jre8
MAINTAINER Eddie Ballisty, eddie.ballisty@blueriver.com

EXPOSE 8888

ENV LUCEE_JARS_URL http://release.lucee.org/rest/update/provider/loader/5.2.1.9
ENV LUCEE_JAVA_OPTS "-Xms1024m -Xmx1024m"

# Download core JAR, and delete it in one step to avoid committing the installer in a FS layer
RUN wget -nv $LUCEE_JARS_URL -O /root/lucee.jar && \
	mkdir -p /usr/local/tomcat/lucee && \
	cp /root/lucee.jar /usr/local/tomcat/lucee/lucee.jar && \
	rm -rf /root/lucee.jar

# Delete the default Tomcat webapps so they aren't deployed at startup
RUN rm -rf /usr/local/tomcat/webapps/*

# Set Tomcat config to load Lucee
COPY catalina.properties server.xml web.xml /usr/local/tomcat/conf/

# Custom setenv.sh to load Lucee
COPY setenv.sh /usr/local/tomcat/bin/
RUN chmod a+x /usr/local/tomcat/bin/setenv.sh

# Create Lucee configs
COPY lucee-server.xml /opt/lucee/server/lucee-server/context/
COPY lucee-web.xml.cfm /opt/lucee/web/

# extensions
COPY s3-extension-0.9.4.113.lex /opt/lucee/server/lucee-server/deploy/
COPY mongodb-extension-3.2.2.54.lex /opt/lucee/server/lucee-server/deploy/
COPY extension-loganalyzer-2.3.1.16.lex /opt/lucee/server/lucee-server/deploy/
COPY ojdbc7-12.1.0.2.lex /opt/lucee/server/lucee-server/deploy/
COPY extension-memcached-3.0.2.29.lex /opt/lucee/server/lucee-server/deploy/

# urlrewrite
COPY urlrewritefilter-4.0.3.jar /usr/local/tomcat/lib/

# Provide test page
RUN mkdir -p /var/www
COPY index.cfm /var/www/
ONBUILD RUN rm -rf /var/www/*

# lucee first time startup; explodes lucee and installs bundles/extensions
RUN /usr/local/tomcat/bin/catalina.sh start && \
    while [ ! -f "/opt/lucee/web/logs/application.log" ] ; do sleep 60; done && \
    /usr/local/tomcat/bin/catalina.sh stop

# Replace the Trusted SSL Certificates packaged with Lucee with those from Debian
#   ca-certificates package from the OS is the most recent authority
RUN cp -f /etc/ssl/certs/java/cacerts /opt/lucee/server/lucee-server/context/security/cacerts
