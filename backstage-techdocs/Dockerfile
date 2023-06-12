FROM debian:bullseye-slim

# renovate: datasource=node-version depName=node versioning=npm
ARG NODE_VERSION=v16.20.0
# renovate: datasource=github-release-attachments depName=plantuml/plantuml
ARG PLANTUML_VERSION=v1.2023.7
# renovate: datasource=java-version depName=java packageName=java-jre
ARG JAVA_VERSION=17.0.7+7
# Arch is provided by buildx
ARG TARGETPLATFORM
ARG TARGETARCH
ARG TARGETVARIANT
RUN printf "I'm building for TARGETPLATFORM=${TARGETPLATFORM}" \
    && printf ", TARGETARCH=${TARGETARCH}" \
    && printf ", TARGETVARIANT=${TARGETVARIANT}"

COPY requirements.txt /tmp/requirements.txt

LABEL org.opencontainers.image.source="https://github.com/moving-shock/containers" \
      org.opencontainers.image.base.name="ghcr.io/moving-shock/backstage-techdocs" \
      org.opencontainers.image.description="Backstage:Techdocs Container" \
      org.opencontainers.image.title="techdocs" 

USER root

# Install prerequisites
RUN apt-get update && \
    apt-get install -y software-properties-common ca-certificates apt-transport-https gnupg wget curl graphviz fontconfig

# Install python
RUN apt install -y python3 python3-dev python3-pip && \
    pip install --upgrade pip

# Install Temurin Java
# Get the major version of Java
RUN echo ${JAVA_VERSION} | grep -o '^[0-9]*' > /tmp/java_major_version.txt
# Add public key
RUN wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | apt-key add -
# Add repository to the list of sources
RUN echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
# update the package sources
RUN apt-get update
# install Azul Zulu JDK 17
RUN JAVA_MAJOR_VERSION=$(echo "${JAVA_VERSION}" | grep -o '^[0-9]*') \
 && apt-get --yes install temurin-${JAVA_MAJOR_VERSION}-jre=${JAVA_VERSION}

# Install node
RUN NODE_ARCH=$(if [ "${TARGETARCH}" = "amd64" ]; then echo "x64"; else echo "${TARGETARCH}"; fi) \
 && cd /opt \
 && curl -v -LO https://nodejs.org/dist/${NODE_VERSION}/node-${NODE_VERSION}-linux-${NODE_ARCH}.tar.xz \
 && tar xJf node-${NODE_VERSION}-linux-${NODE_ARCH}.tar.xz --transform "s/node-${NODE_VERSION}-linux-${NODE_ARCH}/node-${NODE_VERSION}-linux/" \
 && rm node-${NODE_VERSION}-linux-${NODE_ARCH}.tar.xz
ENV PATH=/opt/node-${NODE_VERSION}-linux/bin:${PATH}

# Download plantuml file, Validate checksum & Move plantuml file
RUN curl -o plantuml.jar -L https://github.com/plantuml/plantuml/releases/download/${PLANTUML_VERSION}/plantuml.jar \
    && mv plantuml.jar /opt/plantuml.jar

RUN echo '#!/bin/sh\n\njava -jar '/opt/plantuml.jar' ${@}' >> /usr/local/bin/plantuml
RUN chmod 755 /usr/local/bin/plantuml

# Install mkdocs
RUN pip install -r /tmp/requirements.txt

# Install techdocs cli
RUN npm install -g @techdocs/cli

WORKDIR /content

# Cleanup
RUN rm -rf /var/cache/apt/* && \
    rm -rf /tmp/setup
