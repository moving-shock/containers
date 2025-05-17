FROM debian:12-slim@sha256:4b50eb66f977b4062683ff434ef18ac191da862dbe966961bc11990cf5791a8d

ARG USERNAME=techdocs
ARG USER_UID=1001
ARG USER_GID=$USER_UID
# renovate: datasource=node-version depName=node versioning=npm
ARG NODE_VERSION=v22.15.1
# renovate: datasource=github-release-attachments depName=plantuml/plantuml
ARG PLANTUML_VERSION=v1.2025.2
# renovate: datasource=java-version depName=java packageName=java-jre
ARG JAVA_VERSION=17.0.9.0.0+9
# renovate: datasource=npm depName=@techdocs/cli
ARG TECHDOCS_VERSION=1.9.2
# Arch is provided by buildx
ARG TARGETPLATFORM
ARG TARGETARCH
ARG TARGETVARIANT
RUN printf "I'm building for TARGETPLATFORM=%s" "${TARGETPLATFORM}" \
    && printf ", TARGETARCH=%s" "${TARGETARCH}" \
    && printf ", TARGETVARIANT=%s" "${TARGETVARIANT}"



LABEL org.opencontainers.image.source="https://github.com/moving-shock/containers" \
      org.opencontainers.image.base.name="ghcr.io/moving-shock/backstage-techdocs" \
      org.opencontainers.image.description="Backstage:Techdocs Container" \
      org.opencontainers.image.title="techdocs" 

USER root

# Install prerequisites
RUN apt-get update \
    && apt-get install -y software-properties-common ca-certificates apt-transport-https gnupg curl graphviz fontconfig \
    && apt-get install -y python3 python3-pip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Update pip
# hadolint ignore=DL3013
RUN pip install --break-system-packages --upgrade --no-cache-dir pip

# Install Temurin Java
# Get the major version of Java
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN echo ${JAVA_VERSION} | grep -o '^[0-9]*' > /tmp/java_major_version.txt
# Add public key
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN curl -o - https://packages.adoptium.net/artifactory/api/gpg/key/public | apt-key add -
# Add repository to the list of sources
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
# install Azul Zulu JDK 17
RUN JAVA_MAJOR_VERSION=$(echo "${JAVA_VERSION}" | grep -o '^[0-9]*') \
  && apt-get update \
  && apt-get --yes install temurin-${JAVA_MAJOR_VERSION}-jre=${JAVA_VERSION}  \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

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
COPY requirements.txt /tmp/requirements.txt
RUN pip install --break-system-packages --no-cache-dir -r /tmp/requirements.txt

# Install techdocs cli
RUN npm install -g @techdocs/cli@${TECHDOCS_VERSION}

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
  && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME

USER $USERNAME

WORKDIR /content
