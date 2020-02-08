FROM adoptopenjdk/openjdk11 AS jdkBuilder

RUN $JAVA_HOME/bin/jlink \
\
--module-path /opt/jdk/jmods \
\
--verbose \
\
--add-modules java.base,java.logging,java.xml,java.sql,jdk.unsupported,java.naming,java.desktop,java.management,java.security.jgss,java.security.sasl,jdk.crypto.cryptoki,jdk.crypto.ec,java.instrument,jdk.management.agent,jdk.localedata \
\
--output /opt/jdk-minimal \
\
--compress 2 \
\
--no-header-files



FROM debian:9-slim

COPY --from=jdkBuilder /opt/jdk-minimal /opt/jdk-minimal

ENV JAVA_HOME=/opt/jdk-minimal

COPY Dockerfile target/*.jar /opt/

CMD $JAVA_HOME/bin/java $JAVA_OPTS -jar /opt/*.jar



RUN apt-get clean && apt-get update && apt-get upgrade -y \
&& apt-get install -y --no-install-recommends curl ca-certificates \
&& rm -rf /var/lib/apt/lists/*


ENV TOMCAT_VERSION 8.5.50
ENV IIQ_VERSION 7.2

# Fix sh
#
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

RUN apt-get update && \
apt-get install -y apt-utils wget unzip tar mysql-client

# Get Tomcat
#
RUN wget --quiet --no-cookies http://www-eu.apache.org/dist/tomcat/tomcat-8/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz -O /tmp/tomcat.tgz && \
tar xzvf /tmp/tomcat.tgz -C /opt && \
mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat && \
rm /tmp/tomcat.tgz && \
rm -rf /opt/tomcat/webapps/examples && \
rm -rf /opt/tomcat/webapps/docs && \
rm -rf /opt/tomcat/webapps/ROOT

# Add admin/admin user
#
ADD tomcat-users.xml /opt/tomcat/conf/
run mkdir -p /opt/tomcat/conf/Catalina/localhost
ADD manager.xml /opt/tomcat/conf/Catalina/localhost

# add IIQ
#
COPY src/identityiq-${IIQ_VERSION}.zip /tmp
RUN unzip /tmp/identityiq-${IIQ_VERSION}.zip identityiq.war && \
mkdir /opt/tomcat/webapps/identityiq && \
unzip identityiq.war -d /opt/tomcat/webapps/identityiq && \
chmod +x /opt/tomcat/webapps/identityiq/WEB-INF/bin/iiq && \
rm identityiq.war

RUN mkdir /opt/tomcat/webapps/ROOT
COPY index.html /opt/tomcat/webapps/ROOT

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENV CATALINA_HOME /opt/tomcat
ENV PATH $PATH:$CATALINA_HOME/bin

EXPOSE 8080
EXPOSE 8009
VOLUME "/opt/tomcat/webapps"
WORKDIR /opt/tomcat

# Launch IIQ
#
CMD ["/entrypoint.sh", "run"]
#CMD ["/opt/tomcat/bin/catalina.sh", "run"]

