FROM ubuntu:14.04
MAINTAINER lyysdy@foxmail.com

USER root

# install dev tools
RUN apt-get update \
    && apt-get install -qqy \
       curl \
       tar \
       g++-4.8 \
       gcc \
       libtool \
       pkg-config \
       autoconf \
       openssh-server \
       openssh-client \
       rsync \
       build-essential \
       automake \
       vim \
       gdb \
       git

# Cmake
RUN curl -sc 0 https://cmake.org/files/v3.4/cmake-3.4.1-Linux-x86_64.tar.gz | tar -xz -C /usr/local  \
    && cd /usr/local \
    && ln -s ./cmake-3.4.1-Linux-x86_64 cmake
ENV PATH $PATH:/usr/local/cmake/bin

# open-mpi
RUN wget -q -O - https://www.open-mpi.org/software/ompi/v1.10/downloads/openmpi-1.10.1.tar.gz | tar -xzf - && \
    cd openmpi-1.10.1 && \
    ./configure --prefix=/usr/local/mpi && \
    make -j"$(nproc)" install && \
    rm -rf /openmpi-1.10.1

ENV PATH /usr/local/mpi/bin:$PATH
ENV LD_LIBRARY_PATH /usr/local/mpi/lib:$LD_LIBRARY_PATH

# java
RUN mkdir -p /usr/local/java/default && \
    curl -Ls 'http://download.oracle.com/otn-pub/java/jdk/8u65-b17/jdk-8u65-linux-x64.tar.gz' -H 'Cookie: oraclelicense=accept-securebackup-cookie' | \
    tar --strip-components=1 -xz -C /usr/local/java/default/

ENV JAVA_HOME /usr/local/java/default/ 
ENV PATH $PATH:$JAVA_HOME/bin

# hadoop
RUN wget -cq -t 0 http://www.eu.apache.org/dist/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz 
RUN tar -xz -C /usr/local/ -f hadoop-2.6.0.tar.gz \
    && rm hadoop-2.6.0.tar.gz \
    && cd /usr/local && ln -s ./hadoop-2.6.0 hadoop \
    && cp -r /usr/local/hadoop/include/* /usr/local/include

ENV HADOOP_PREFIX /usr/local/hadoop
RUN sed -i '/^export JAVA_HOME/ s:.*:export JAVA_HOME=/usr/local/java/default\nexport HADOOP_PREFIX=/usr/local/hadoop\nexport HADOOP_HOME=/usr/local/hadoop\n:' $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh
RUN sed -i '/^export HADOOP_CONF_DIR/ s:.*:export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop/:' $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh

# fixing the libhadoop.so like a boss
RUN rm  /usr/local/hadoop/lib/native/* \
    && curl -Ls http://dl.bintray.com/sequenceiq/sequenceiq-bin/hadoop-native-64-2.6.0.tar | tar -x -C /usr/local/hadoop/lib/native/


WORKDIR /dmtk
RUN cd /dmtk && git clone git@github.com:Microsoft/multiverso.git . && \
    cd multiverso && \
    make -j4 all
RUN cd /dmtk && git clone git@github.com:Microsoft/distributed_word_embedding.git.git . && \
    cd distributed_word_embedding && \
    ln /dmtk/multiverso multiverso && \
    make -j4
RUN cd /dmtk && git clone git@github.com:Microsoft/distributed_skipgram_mixture.git.git . && \
    cd distributed_skipgram_mixture && \
    ln /dmtk/multiverso multiverso && \
    make -j4
RUN cd /dmtk && git clone git@github.com:Microsoft/lightlda.git.git . && \
    cd lightlda && \
    ln /dmtk/multiverso multiverso && \
    make -j4
   
# clean unnessary packages
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV PATH /dmtk/distributed_word_embedding/bin:/dmtk/distributed_skipgram_mixture/bin:/dmtk/lightlda/bin:$PATH
