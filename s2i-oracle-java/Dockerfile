FROM rhel:7.2

ENV MAVEN_VERSION="3.3.3" \
    JOLOKIA_VERSION="1.3.5" \
    JAVA_HOME="/usr/lib/jvm/java-1.8.0" \
   # JAVA_VENDOR="openjdk" \
   # JAVA_VERSION="1.8.0" \
    JAVA_VERSION 8u45 \
    BUILD_VERSION b14 \
    PATH=$PATH:"/usr/local/s2i" \
    AB_JOLOKIA_AUTH_OPENSHIFT="true" \
    AB_JOLOKIA_PASSWORD_RANDOM="true" \
    JAVA_DATA_DIR=/deployments/data

# Some version information
LABEL io.k8s.description="Platform for building and running plain Java applications" \
      io.k8s.display-name="Java Applications" \
      io.openshift.tags="builder,java" \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/tmp" \
      io.openshift.expose-services="8080,8778" \
      org.jboss.deployments-dir="/deployments"

# Install JDK
RUN yum install -y wget && \
    wget --no-cookies --no-check-certificate \
        --header "Cookie: oraclelicense=accept-securebackup-cookie" \
        "http://download.oracle.com/otn-pub/java/jdk/$JAVA_VERSION-$BUILD_VERSION/jdk-$JAVA_VERSION-linux-x64.rpm" \
        -O /tmp/jdk-8-linux-x64.rpm && \
    yum install -y /tmp/jdk-8-linux-x64.rpm && \
    yum remove -y wget && \
    yum clean all && \
    rm -fr /tmp/jdk-8-linux-x64.rpm
    
    # Add Java User
    groupadd -r java -g 1000 && \
    useradd -u 185 -r -g root -m -d /opt/java -s /sbin/nologin -c "Java user" java && \
    usermod -a -G java java && \
    
    # Install Maven
    curl https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz | \
    tar -xzf - -C /opt && \
    ln -s /opt/apache-maven-${MAVEN_VERSION} /opt/maven && \
    ln -s /opt/maven/bin/mvn /usr/bin/mvn && \
    
    # Install Jolokia
    mkdir -p /opt/jolokia && \
    curl http://central.maven.org/maven2/org/jolokia/jolokia-jvm/${JOLOKIA_VERSION}/jolokia-jvm-${JOLOKIA_VERSION}-agent.jar \
         -o /opt/jolokia/jolokia.jar && \
         
    echo securerandom.source=file:/dev/urandom >> /usr/lib/jvm/java/jre/lib/security/java.security


ADD jolokia-opts /opt/jolokia/jolokia-opts
ADD jolokia.properties /opt/jolokia/etc/jolokia.properties

RUN mkdir -p /opt/jolokia/etc && \
    chmod 444 /opt/jolokia/jolokia.jar && \
    chmod 755 /opt/jolokia/jolokia-opts && \
    chmod -R 775 /opt/jolokia/etc && \
    chgrp -R root /opt/jolokia/etc && \
    chown java:root /opt/jolokia/etc/jolokia.properties

COPY run-java.sh run-env.sh debug-options container-limits java-default-options /opt/run-java/
RUN chmod 755 /opt/run-java/run-java.sh /opt/run-java/run-env.sh /opt/run-java/java-default-options /opt/run-java/container-limits /opt/run-java/debug-options

# Expose Ports
EXPOSE 8080 8778

# S2I scripts + README
COPY s2i /usr/local/s2i
RUN chmod 755 /usr/local/s2i/*
ADD README.md /usr/local/s2i/usage.txt

RUN mkdir -p /deployments/data && \
 chmod -R "g+rwX" /deployments && \
 chown -R java:root /deployments
 
COPY settings.xml /opt/java/.m2/settings.xml

# S2I requires a numeric, non-0 UID. This is the UID for the java user in the base image
USER 185

WORKDIR /opt/java

# Use the run script as default since we are working as an hybrid image which can be
# used directly to. (If we were a plain s2i image we would print the usage info here)
CMD [ "/usr/local/s2i/run" ]
