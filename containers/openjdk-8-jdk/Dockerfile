FROM bsantanna/openssh-client
MAINTAINER Bruno Sant'Anna "bruno@btech.software"
ENV REFRESHED_AT 2016-12-29
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/

RUN apt-get update -qq && apt-get install -qqy ca-certificates openjdk-8-jdk

CMD ["/usr/lib/jvm/java-8-openjdk-amd64/bin/java", "-version"]
