# OpenSwitch
#
# VERSION 0.0.1

FROM nhanaue/openswitch:base

MAINTAINER Naoto Hanaue <hanaue.naoto@po.ntts.co.jp>

ADD libraries.tar.gz /usr 

ENV LD_LIBRARY_PATH="/usr/local/lib"
ENV GOROOT="/usr/local/go"
ENV GOPATH="/home/root/go"
ENV PATH="$PATH:$GOROOT/bin:$GOPATH/bin"

WORKDIR /home/root
RUN cp -Rp /usr/local/python/* /usr/lib/python2.7/
RUN wget https://bootstrap.pypa.io/get-pip.py
RUN rm -rf /usr/lib/python2.7/site-packages/distribute*           
RUN rm -rf /usr/lib/python2.7/site-packages/setuptools* 
RUN python get-pip.py

RUN wget https://storage.googleapis.com/golang/go1.5.linux-amd64.tar.gz
RUN tar xzvf go1.5.linux-amd64.tar.gz
RUN rm -rf go1.5.linux-amd64.tar.gz
RUN mv go/ /usr/local/
