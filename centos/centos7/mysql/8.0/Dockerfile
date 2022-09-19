FROM centos:centos7.9.2009 AS build
WORKDIR /usr/local

RUN export VERSION=8.0.28 \
  && curl -O https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-${VERSION}-linux-glibc2.17-x86_64-minimal.tar.xz \
  && xz -d mysql-${VERSION}-linux-glibc2.17-x86_64-minimal.tar.xz \
  && tar -xf mysql-${VERSION}-linux-glibc2.17-x86_64-minimal.tar \
  && mv mysql-${VERSION}-linux-glibc2.17-x86_64-minimal mysql \
  && chown -R nobody.nobody mysql

FROM centos:centos7.9.2009
WORKDIR /
RUN yum install -y libaio numactl && yum clean all
COPY --from=build /usr/local/mysql /usr/local/mysql
ENV PATH /usr/local/mysql/bin/:$PATH
