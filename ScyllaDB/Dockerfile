# © Copyright IBM Corporation 2019, 2020.
# LICENSE: Apache License, Version 2.0 (http://www.apache.org/licenses/LICENSE-2.0)

########## Dockerfile for ScyllaDB version 2.3.1 #########
#
# This Dockerfile builds a basic installation of ScyllaDB.
#
# ScyllaDB is a high performance distributed NoSQL database.
#
# To build this image, from the directory containing this Dockerfile
# (assuming that the file is named Dockerfile):
# docker build --build-arg TARGET=<target_value> -t <image_name> .
#
# To start ScyllaDB Server run the below command:
# Replace the following parameters in the command below:
#   - <container_name> : Name of the container
#   - <image_name>     : Name of the ScyllaDB image
#   - <ip_address>     : IP address of the host machine to access the ScyllaDB server.
#
# docker run \
#       -tid \
#       --name <container_name> \
#       -p 10000:10000 -p 9042:9042 -p 9160:9160 -p 7000:7000 -p 7001:7001 \
#       <image_name> \
#       scylla \
#               --options-file /etc/scylla/scylla.yaml \
#               --max-io-requests 65 \
#               --listen-address <ip_address> \
#               --rpc-address <ip_address> \
#               --seed-provider-parameters seeds=<ip_address>
#
# Reference :
# http://www.scylladb.com
#
##################################################################################

# Base Image
FROM s390x/ubuntu:16.04 AS builder

ENV SOURCE_DIR=/tmp/source LD_LIBRARY_PATH=/opt/gcc-7.4.0/lib64:$LD_LIBRARY_PATH PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/
ENV ANT_HOME=$SOURCE_DIR/ant/apache-ant-1.10.6 
ENV PATH=/opt/gcc-7.4.0/bin:/usr/local/bin:$ANT_HOME/bin:$PATH

ARG TARGET

WORKDIR $SOURCE_DIR

# Install dependencies
RUN apt-get update && apt-get install -y openjdk-8-jdk python libgnutls-dev systemtap-sdt-dev lksctp-tools \
        xfsprogs snappy libyaml-dev maven cmake openssl perl libc-ares-dev libevent-dev libmpfr-dev libmpcdec-dev  \
	xz-utils automake gcc git make texinfo wget unzip libtool libssl-dev curl libsystemd-dev libhwloc-dev \
	libaio-dev libsctp-dev libsnappy-dev libpciaccess-dev libxml2-dev xfslibs-dev libgnutls28-dev libiconv-hook-dev \
	mpi-default-dev libbz2-dev python-dev libxslt-dev libjsoncpp-dev cmake ragel python3 python3-pyparsing \
	libprotobuf-dev protobuf-compiler liblz4-dev ninja-build libcrypto++-dev python-pystache python-setuptools checkinstall \
# Install dependency : GCC 7.4.0
        && cd $SOURCE_DIR \
        && mkdir gcc && cd gcc \
        && curl -SLO https://ftpmirror.gnu.org/gcc/gcc-7.4.0/gcc-7.4.0.tar.xz \
        && tar -xf gcc-7.4.0.tar.xz && cd gcc-7.4.0 \
        && ./contrib/download_prerequisites \
        && mkdir objdir && cd objdir \
        && ../configure --prefix=/opt/gcc-7.4.0 --enable-languages=c,c++ --with-arch=zEC12 --with-long-double-128 \
             --build=s390x-linux-gnu --host=s390x-linux-gnu --target=s390x-linux-gnu                  \
             --enable-threads=posix --with-system-zlib --disable-multilib  \
        && make -j 8 \
        && cd s390x-linux-gnu/libstdc++-v3 \
        && checkinstall -y --pkgversion=3 \
        && cd ../.. \
        && make install \
        && ln -sf /opt/gcc-7.4.0/bin/gcc /usr/bin/gcc \
        && ln -sf /opt/gcc-7.4.0/bin/g++ /usr/bin/g++ \
# Install dependency : ant
        && cd $SOURCE_DIR \
        && mkdir ant && cd ant \
        && curl -SLO https://mirrors.estointernet.in/apache/ant/binaries/apache-ant-1.10.6-bin.tar.gz \
        && tar -xzf apache-ant-1.10.6-bin.tar.gz \
        && ant -version \
# Install dependency : antlr3
        && cd $SOURCE_DIR \
        && mkdir antlr3 && cd antlr3 \
        && curl -SLO https://github.com/antlr/antlr3/archive/3.5.2.tar.gz \
        && tar -xzf 3.5.2.tar.gz && cd antlr3-3.5.2 \
        && cp runtime/Cpp/include/antlr3* /usr/local/include/ \
        && cd antlr-complete \
        && MAVEN_OPTS="-Xmx4G" mvn \
        && echo 'java -cp '"$(pwd)"'/target/antlr-complete-3.5.2.jar org.antlr.Tool $@' | tee /usr/local/bin/antlr3 \
        && chmod +x /usr/local/bin/antlr3 \
# Install dependency : Boost
        && cd $SOURCE_DIR \
        && mkdir boost && cd boost \
        && curl -SLO https://dl.bintray.com/boostorg/release/1.68.0/source/boost_1_68_0.tar.gz \
        && tar -xf boost_1_68_0.tar.gz && cd boost_1_68_0 \
        && sed -i 's/array\.hpp/array_wrapper.hpp/g' boost/numeric/ublas/matrix.hpp \
        && sed -i 's/array\.hpp/array_wrapper.hpp/g' boost/numeric/ublas/storage.hpp \
        && ./bootstrap.sh \
        && ./b2 toolset=gcc variant=release link=static runtime-link=static threading=multi cxxstd=14 --prefix=/usr/local/ --without-python install \
# Install dependency : Thrift
        && cd $SOURCE_DIR \
        && mkdir thrift && cd thrift \
        && curl -SLO http://archive.apache.org/dist/thrift/0.9.3/thrift-0.9.3.tar.gz \
        && tar -xzf thrift-0.9.3.tar.gz \
        && cd thrift-0.9.3 \
        && ./configure --without-lua --without-go \
        && make -j 8 \
        && make install \
# Install dependency : yaml-cpp
        && cd $SOURCE_DIR \
        && mkdir yaml-cpp && cd yaml-cpp \
        && curl -SLO https://github.com/jbeder/yaml-cpp/archive/yaml-cpp-0.6.2.tar.gz \
        && tar -xzf yaml-cpp-0.6.2.tar.gz \
        && mkdir yaml-cpp-yaml-cpp-0.6.2/build && cd yaml-cpp-yaml-cpp-0.6.2/build \
        && cmake .. \
        && make && make install \
# Build ScyllaDB
        && cd $SOURCE_DIR \
        && git clone -b branch-2.3-s390x https://github.com/linux-on-ibm-z/scylla \
        && cd scylla \
        && git submodule update --init --recursive \
        && ./configure.py --mode=release --target=$TARGET --debuginfo 1 --static --static-boost --static-thrift \
        && ninja -j 8 \
        && checkinstall -y --pkgversion=2.3.1 ./install.sh
		
FROM s390x/ubuntu:16.04

LABEL maintainer="LoZ Open Source Ecosystem (https://www.ibm.com/developerworks/community/groups/community/lozopensource)"

ENV LD_LIBRARY_PATH=/opt/gcc-7.4.0/lib64:$LD_LIBRARY_PATH 
ENV SOURCE_DIR=/tmp/source

WORKDIR $SOURCE_DIR

COPY --from=builder $SOURCE_DIR/scylla/scylla_2.3.1-1_s390x.deb $SOURCE_DIR/
COPY --from=builder $SOURCE_DIR/gcc/gcc-7.4.0/objdir/s390x-linux-gnu/libstdc++-v3/libstdc++_3-1_s390x.deb $SOURCE_DIR/

RUN apt-get update && apt-get install -y \
                libcrypto++9v5 \
                libgnutls30 \ 
                libgnutlsxx28 \
                libprotobuf9v5 \ 
                libatomic1 \
                libsnappy1v5 \ 
                libjsoncpp1 \
        && apt install ./libstdc++_3-1_s390x.deb \
        && apt install ./scylla_2.3.1-1_s390x.deb \
        && rm -rf $SOURCE_DIR

WORKDIR /root
 
EXPOSE 10000 9042 9160 7000 7001

CMD scylla
