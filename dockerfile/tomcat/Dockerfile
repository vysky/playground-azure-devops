# FROM tomcat:9.0.62-jdk11

FROM ubuntu:20.04

# define default command
CMD bash

# apt and apt-get
# apt is for terminal (will show warning if run docker build using apt instead of apt-get)
# apt-get is for script
RUN apt-get update
RUN apt-get -y upgrade
# wget - to download tomcat
# curl - use by tomcat
RUN apt-get -y install openjdk-11-jdk wget curl

RUN mkdir /usr/local/tomcat
RUN wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.68/bin/apache-tomcat-9.0.68.tar.gz -O /tmp/tomcat.tar.gz
WORKDIR /tmp
RUN tar xvfz tomcat.tar.gz
RUN cp -Rv apache-tomcat-9.0.68/* /usr/local/tomcat/

EXPOSE 8080
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64

WORKDIR /usr/local/tomcat/
# RUN cp -r webapps.dist/manager webapps/manager
# RUN cp -r webapps.dist/host-manager webapps/host-manager

COPY tomcat-users.xml conf/tomcat-users.xml
COPY context.xml webapps/manager/META-INF/context.xml
COPY context.xml webapps/host-manager/META-INF/context.xml

CMD ["bin/catalina.sh", "run"]