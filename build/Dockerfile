FROM ubuntu:16.04

MAINTAINER Elliot Metsger <emetsger@jhu.edu>
LABEL description = "Provides a Java 8 JDK, Git, Maven, and a seeded Maven repository under ${MAVEN_REPO}"

# Update repository contents and add base packages

RUN apt-get update && \
    apt-get -y install \
      default-jdk \
      maven \
      git \
      bash \
      curl \
      gettext \
      net-tools

# Set JAVA_HOME

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

ENV DEBUG          false
ENV DEBUG_PORT     5005
ENV DEBUG_ARG      "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=\${DEBUG_PORT}"

# Prepare the Maven repository by seeding some commonly used artifacts

ENV MAVEN_REPO /build/repository

RUN mkdir -p ${MAVEN_REPO}

WORKDIR /tmp

RUN \

    # Simple project used for mvn commands below
    echo "<project> <modelVersion>4.0.0</modelVersion> <groupId>foo</groupId> <artifactId>bar</artifactId> <version>1.0-SNAPSHOT</version> </project>" > pom.xml && \

    # Seed various artifacts likely to be used, saving some time when running this container.
    mvn org.apache.maven.plugins:maven-clean-plugin:3.0.0:clean && \
    mvn org.apache.maven.plugins:maven-compiler-plugin:3.5.1:compile && \
    mvn org.apache.maven.plugins:maven-install-plugin:2.5.2:install -Dskip || true && \
    mvn org.apache.maven.plugins:maven-surefire-plugin:2.19.1:test && \
    mvn org.apache.maven.plugins:maven-verifier-plugin:1.1:verify || true && \
    rm pom.xml               && \
    rm -rf target/           && \
    rm -rf ${MAVEN_REPO}/foo

# Alias the `mvn` command to `mvn -Dmaven.repo.local=${MAVEN_REPO}`
# The alias is created at container runtime so that child images can override
# the ${MAVEN_REPO} environment variable

RUN \
  echo "eval 'alias mvn=\"mvn -Dmaven.repo.local=\${MAVEN_REPO}\"'" \
    >> /etc/skel/.bashrc                                                 && \
  echo "eval 'alias mvn=\"mvn -Dmaven.repo.local=\${MAVEN_REPO}\"'" \
    >> ~/.bashrc
