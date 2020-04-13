FROM openjdk:8-jre-slim AS builder

MAINTAINER mark.prichard@appdynamics.com

ENV APPD_AGENT_VERSION 20.3.2.2520
# after adding the version to MachineAgent.zip -> MachineAgent-20.3.2.2520.zip
# run the command: sha256sum MachineAgent-20.3.2.2520.zip
ENV APPD_AGENT_SHA256 f9f92a71dc023743ca409806f0909e5ed0219a203add4ec78dc262c4cf6e1c65

ARG APPD_AGENT_VERSION
ARG APPD_AGENT_SHA256

# need to install base applications first to prevent errors:
# /bin/sh: 1: unzip: not found
RUN apt-get update && apt-get -y upgrade && apt-get install -y unzip && apt-get install -y apt-utils \
    && apt-get install -y iproute2 && apt-get install -y procps && apt-get install -y sysstat && apt-get install -y git \
    && apt-get install -y python && apt-get install -y curl && apt-get install -y zip

COPY MachineAgent-${APPD_AGENT_VERSION}.zip /
RUN echo "${APPD_AGENT_SHA256} *MachineAgent-${APPD_AGENT_VERSION}.zip" >> appd_checksum \
    && sha256sum -c appd_checksum \
    && rm appd_checksum \
    && unzip -oq MachineAgent-${APPD_AGENT_VERSION}.zip -d /tmp


FROM openjdk:8-jre-slim
RUN apt-get update && apt-get -y upgrade && apt-get install -y unzip && apt-get install -y apt-utils \
    && apt-get install -y iproute2 && apt-get install -y procps && apt-get install -y sysstat && apt-get install -y git \
    && apt-get install -y python && apt-get install -y curl

COPY --from=builder /tmp /opt/appdynamics

ENV MACHINE_AGENT_HOME /opt/appdynamics/
ENV KUBERNETES_EXTENSION_HOME /

##download and setup k8's extension

ADD KubernetesEventsMonitor ${MACHINE_AGENT_HOME}/monitors/KubernetesEventsMonitor

##add copy of log4j.xml here and k8s extension config and GKE kube config
COPY log4j.xml /opt/appdynamics/conf/logging/

COPY config /.kube/config

ADD start-appdynamics ${MACHINE_AGENT_HOME}
RUN chmod 744 ${MACHINE_AGENT_HOME}/start-appdynamics

#RUN curl https://sdk.cloud.google.com / | bash

# Configure and Run AppDynamics Machine Agent
CMD "${MACHINE_AGENT_HOME}/start-appdynamics"

#CMD ["sh", "-c", "java ${MACHINE_AGENT_PROPERTIES} -jar /opt/appdynamics/machineagent.jar"]
