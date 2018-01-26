FROM registry.access.redhat.com/rhscl/s2i-base-rhel7

ENV MAVEN_VERSION="3.3.3" \
  # JOLOKIA_VERSION="1.3.5" \
  # HOME=/opt/app-root/src \
    JAVA_HOME=/opt/app-root/src/jdk1.8.0_102 \
    PATH=$PATH:"/usr/local/s2i":$JAVA_HOME/bin \
    JAVA_VERSION="1.8.0" \
  # AB_JOLOKIA_AUTH_OPENSHIFT="true" \
  # AB_JOLOKIA_PASSWORD_RANDOM="true" \
   JAVA_DATA_DIR=/deployments/data

RUN echo $PATH

# Some version information
LABEL io.k8s.description="Platform for building and running plain Java applications" \
      io.k8s.display-name="Java Applications" \
      io.openshift.tags="builder,java" \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/tmp" \
      io.openshift.expose-services="8080,8778" \
      org.jboss.deployments-dir="/deployments"

RUN wget http://durahc1mn01-stg.corp.netapp.com/extra-packages/java/jdk-8u102-linux-x64.tar.gz && \
    ls -ltr && tar -zxvf jdk-8u102-linux-x64.tar.gz && \
    ls -ltr && rm jdk-8u102-linux-x64.tar.gz && \
 
   # rm /usr/lib/jvm/jdk-8u102-linux-x64.tar.gz && \
    
   # Add Java User
   # groupadd -r java -g 1000 && \
   # useradd -u 185 -r -g root -m -d /opt/java -s /sbin/nologin -c "Java user" java && \
   # usermod -a -G java java && \
    
    # Install Maven
    curl https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz | \
    tar -xzf - -C /opt && \
    ln -s /opt/apache-maven-${MAVEN_VERSION} /opt/maven && \
    ln -s /opt/maven/bin/mvn /usr/bin/mvn
    
         
    # echo securerandom.source=file:/dev/urandom >> /usr/lib/jvm/java/jre/lib/security/java.security

COPY run-java.sh run-env.sh debug-options container-limits java-default-options /opt/run-java/
RUN chmod 755 /opt/run-java/run-java.sh /opt/run-java/run-env.sh /opt/run-java/java-default-options /opt/run-java/container-limits /opt/run-java/debug-options

# Expose Ports
EXPOSE 8080 8778

# S2I scripts + README
COPY s2i /usr/local/s2i
RUN chmod 755 /usr/local/s2i/*
ADD README.md /usr/local/s2i/usage.txt

RUN mkdir -p /deployments/data && \
 chmod -R "g+rwX" /deployments
# chown -R java:root /deployments
 
#COPY settings.xml /opt/java/.m2/settings.xml

# S2I requires a numeric, non-0 UID. This is the UID for the java user in the base image
USER 185

WORKDIR $HOME

# Use the run script as default since we are working as an hybrid image which can be
# used directly to. (If we were a plain s2i image we would print the usage info here)
CMD [ "/usr/local/s2i/run" ]
