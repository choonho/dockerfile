FROM ubuntu:14.04
MAINTAINER Choonho Son <choonho.son@gmail.com>

RUN apt-get update
RUN apt-get -y install wget unzip openjdk-7-jre supervisor
RUN wget -O /usr/local/bin/0.4.2.c3.zip https://github.com/intel-cloud/cosbench/releases/download/v0.4.2.c3/0.4.2.c3.zip
RUN cd /usr/local/bin/; unzip 0.4.2.c3.zip;
RUN ln -s /usr/local/bin/0.4.2.c3 /usr/local/bin/cosbench
