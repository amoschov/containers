FROM bsantanna/ubuntu-multi:14.04
MAINTAINER Bruno Sant'Anna "bruno@btech.software"
ENV REFRESHED_AT 2016-12-29
ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64/

RUN apt-get update -qq && apt-get install -qqy ca-certificates openjdk-7-jdk

CMD ["/usr/lib/jvm/java-7-openjdk-amd64/bin/java", "-version"]
