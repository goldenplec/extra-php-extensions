ARG PHP_VERSION
FROM bref/build-php-$PHP_VERSION AS ext

# Datastax is providing a PHP Cassandra driver, but this hasn't been updated for a while and doesn't work with PHP 7.2 or higher
# Datastax indicated there will be updates in the near future, so check https://docs.datastax.com/en/developer/php-driver/ if you want to avoid building your own :-)

ENV LIBUV_VERSION 1.35.0
ENV CCP_DRIVER_VERSION 2.15.2

# Fix library path
ENV LD_LIBRARY_PATH=/usr/lib:/usr/lib64:$LD_LIBRARY_PATH

WORKDIR /tmp

RUN yum install -y automake cmake gcc gcc-c++ git libtool openssl-devel wget gmp gmp-devel boost php-devel pcre-devel

# Install dependencies
RUN curl -o libuv.rpm https://downloads.datastax.com/cpp-driver/centos/7/dependencies/libuv/v$LIBUV_VERSION/libuv-$LIBUV_VERSION-1.el7.x86_64.rpm
RUN yum install -y libuv.rpm
RUN curl -o libuv-dev.rpm https://downloads.datastax.com/cpp-driver/centos/7/dependencies/libuv/v$LIBUV_VERSION/libuv-devel-$LIBUV_VERSION-1.el7.x86_64.rpm
RUN yum install -y libuv-dev.rpm

RUN yum install -y https://downloads.datastax.com/cpp-driver/centos/7/cassandra/v$CCP_DRIVER_VERSION/cassandra-cpp-driver-$CCP_DRIVER_VERSION-1.el7.x86_64.rpm
RUN yum install -y https://downloads.datastax.com/cpp-driver/centos/7/cassandra/v$CCP_DRIVER_VERSION/cassandra-cpp-driver-devel-$CCP_DRIVER_VERSION-1.el7.x86_64.rpm

RUN cd /tmp && \
    git clone https://github.com/datastax/php-driver.git && \
    cd php-driver && \
    pecl install ext/package.xml

RUN cp `php-config --extension-dir`/cassandra.so /tmp/cassandra.so

# Build the final image from the lambci image that is close to the production environment
FROM scratch


# php extension binary
COPY --from=ext /tmp/cassandra.so /opt/bref-extra/cassandra.so

COPY --from=ext /usr/lib64/libcassandra.so.2 /opt/bref/lib64/libcassandra.so.2
COPY --from=ext /usr/lib64/libcassandra_static.a /opt/bref/lib64/libcassandra_static.a

COPY --from=ext /usr/include/cassandra.h /opt/bref/include/cassandra.h
COPY --from=ext /usr/include/dse.h /opt/bref/include/dse.h

COPY --from=ext /usr/lib64/pkgconfig/cassandra.pc /opt/bref/lib64/pkgconfig/cassandra.pc
COPY --from=ext /usr/lib64/pkgconfig/cassandra_static.pc /opt/bref/lib64/pkgconfig/cassandra_static.pc

# dependencies
COPY --from=ext /usr/lib64/libuv.so.1 /opt/bref/lib64/libuv.so.1
