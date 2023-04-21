FROM debian:bullseye-slim

ARG NODE_VERSION=v16.19.1
# Arch is provided by buildx
ARG TARGETPLATFORM
ARG TARGETARCH
ARG TARGETVARIANT
RUN printf "I'm building for TARGETPLATFORM=${TARGETPLATFORM}" \
    && printf ", TARGETARCH=${TARGETARCH}" \
    && printf ", TARGETVARIANT=${TARGETVARIANT}"

COPY .nvmrc /tmp/.nvmrc

RUN printf "$(cat /tmp/.nvmrc | tr -cd [:digit:].)"


LABEL org.opencontainers.image.source="https://github.com/moving-shock/containers" \
      org.opencontainers.image.base.name="ghcr.io/moving-shock/backstage-techdocs" \
      org.opencontainers.image.description="Backstage:Techdocs Container" \
      org.opencontainers.image.title="techdocs" 

USER root

# Install prerequisites
#RUN apt-get update && \
#    apt-get install -y software-properties-common ca-certificates gnupg wget curl graphviz fontconfig

# Install python
#RUN apt install -y python3 python3-dev python3-pip && \
#    pip install --upgrade pip

# Install java
#RUN wget -O- https://apt.corretto.aws/corretto.key | apt-key add - && \
#    add-apt-repository 'deb https://apt.corretto.aws stable main' && \
#    apt update && \
#    apt install -y java-11-amazon-corretto-jdk
# add Azul's public key
#RUN sudo apt-key adv \
#  --keyserver hkp://keyserver.ubuntu.com:80 \
#  --recv-keys 0xB1998361219BD9C9
#
## download and install the package that adds
## the Azul APT repository to the list of sources
#RUN curl -O https://cdn.azul.com/zulu/bin/zulu-repo_1.0.0-3_all.deb
#
## install the package
#RUN sudo apt-get install ./zulu-repo_1.0.0-3_all.deb
#
## update the package sources
#RUN sudo apt-get update
#
## install Azul Zulu JDK 17
#RUN sudo apt-get --yes install zulu17-jdk

# Install node
RUN export NODEARCH=$(if [ "${TARGETARCH}" = "amd64" ]; then echo "x86"; else echo "${TARGETARCH}}"; fi) \
 && cd /opt \
 && curl -LO https://nodejs.org/dist/${NODE_VERSION}/node-${NODE_VERSION}-linux-${NODE_ARCH}.tar.xz \
 && tar xJf node-${NODE_VERSION}-linux-${NODE_ARCH}.tar.xz \
 && rm node-${NODE_VERSION}-linux-${NODE_ARCH}.tar.xz
ENV PATH=/opt/node-${NODE_VERSION}-linux-${NODE_ARCH}/bin:${PATH}

# Download plantuml file, Validate checksum & Move plantuml file
RUN curl -o plantuml.jar -L https://github.com/plantuml/plantuml/releases/download/v1.2023.4/plantuml-1.2023.4.jar \
    && echo "08feded8d9e222dae0a741875d20977b4933f44a  plantuml.jar" | sha1sum -c - \
    && mv plantuml.jar /opt/plantuml.jar

RUN echo '#!/bin/sh\n\njava -jar '/opt/plantuml.jar' ${@}' >> /usr/local/bin/plantuml
RUN chmod 755 /usr/local/bin/plantuml

# Install mkdocs
RUN pip install mkdocs-techdocs-core mkdocs-same-dir pymdown-extensions mkdocs-video mkdocs-section-index

# Install techdocs cli
RUN npm install -g @techdocs/cli

WORKDIR /content

# Cleanup
RUN rm -rf /var/cache/apt/* && \
    rm -rf /tmp/setup
