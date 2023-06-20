FROM rockylinux:9.0.20220720 AS build
ENV SEATUNNEL_VERSION 2.3.2
RUN dnf install -y java-1.8.0-openjdk
RUN curl -O -L https://www.apache.org/dyn/closer.lua/seatunnel/${SEATUNNEL_VERSION}/apache-seatunnel-${SEATUNNEL_VERSION}-bin.tar.gz
RUN tar zxf apache-seatunnel-${SEATUNNEL_VERSION}-bin.tar.gz
RUN copy -r apache-seatunnel-${SEATUNNEL_VERSION}-bin /usr/local/seatunnel
RUN cd /usr/local/seatunnel && sh install-plugin.sh ${SEATUNNEL_VERSION}

FROM rockylinux:9.0.20220720-minimal
RUN microdnf install -y java-1.8.0-openjdk
COPY --from=build /usr/local/seatunnel /usr/local/seatunnel
ENV PATH /usr/local/seatunnel/bin/:$PATH
CMD ["/usr/local/seatunnel/bin/seatunnel.sh --config /usr/local/seatunnel/config/v2.batch.config.template -e lcoal"]
