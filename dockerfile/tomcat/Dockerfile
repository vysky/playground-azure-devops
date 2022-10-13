FROM tomcat:9.0.62-jdk11

WORKDIR /usr/local/tomcat/
RUN cp -r webapps.dist/manager webapps/manager
RUN cp -r webapps.dist/host-manager webapps/host-manager

COPY tomcat-users.xml conf/tomcat-users.xml
COPY context.xml webapps/manager/META-INF/context.xml
COPY context.xml webapps/host-manager/META-INF/context.xml