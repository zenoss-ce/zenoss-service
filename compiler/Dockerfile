FROM ubuntu:bionic

RUN apt-get update && \
    apt-get install -y apt-transport-https ca-certificates curl software-properties-common python-selinux && \
    curl https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
    curl https://download.tuxfamily.org/ach/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] https://download.tuxfamily.org/ach/ubuntu $(lsb_release -cs) main" && \
    apt-get update && \
    apt-get -y install curl debsigs debsig-verify gcc make rpm rpm2cpio ruby ruby-dev serviced && \
    apt-get clean autoclean && \
    apt-get autoremove -y && \
    rm -rf /var/lib/{apt,dpkg,cache,log}/ 

RUN gem install fpm
RUN useradd -u 1000 -m jenkins
