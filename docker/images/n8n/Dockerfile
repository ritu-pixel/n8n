# ==============================================================================
# ARGUMENTS
# ==============================================================================
ARG NODE_VERSION=22
ARG N8N_VERSION=snapshot
ARG LAUNCHER_VERSION=1.1.3

# ==============================================================================
# STAGE 1: System Dependencies & Base Setup
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS system-deps

# Install essential packages
RUN apk add --no-cache bash curl tini python3 make g++ openssl wget

# ==============================================================================
# STAGE 2: Task Runner Launcher
# ==============================================================================
FROM alpine:3.22.0 AS launcher-downloader

ARG LAUNCHER_VERSION
ARG ARCH_NAME=amd64

RUN set -e && \
    apk add --no-cache wget ca-certificates tar openssl && \
    mkdir /launcher-temp && cd /launcher-temp && \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz" && \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256" && \
    echo "$(cat *.sha256) task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz" > checksum.sha256 && \
    sha256sum -c checksum.sha256 && \
    mkdir -p /launcher-bin && \
    tar xzf task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz -C /launcher-bin && \
    rm -rf /launcher-temp

# ==============================================================================
# STAGE 3: Final Runtime Image
# ==============================================================================
FROM system-deps AS runtime

ARG N8N_VERSION
ARG N8N_RELEASE_TYPE=dev

ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE}
ENV NODE_ICU_DATA=/usr/local/lib/node_modules/full-icu
ENV SHELL=/bin/sh

WORKDIR /home/node

# ✅ Install n8n from npm
RUN npm install --global n8n@${N8N_VERSION}

# ✅ Copy launcher binary
COPY --from=launcher-downloader /launcher-bin/* /usr/local/bin/

# ✅ Make sure these exist in your repo!
COPY docker/images/n8n/docker-entrypoint.sh /docker-entrypoint.sh
COPY docker/images/n8n/n8n-task-runners.json /etc/n8n-task-runners.json

# Optional npm fix
RUN npm install -g npm@11.4.2

# Optional PDF fix
RUN cd /usr/local/lib/node_modules/n8n/node_modules/pdfjs-dist && \
    npm install @napi-rs/canvas || true

RUN mkdir -p /home/node/.n8n && chown -R node:node /home/node

EXPOSE 5678/tcp

USER node
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]

LABEL org.opencontainers.image.title="n8n" \
      org.opencontainers.image.description="Workflow Automation Tool" \
      org.opencontainers.image.source="https://github.com/n8n-io/n8n" \
      org.opencontainers.image.url="https://n8n.io" \
      org.opencontainers.image.version=${N8N_VERSION}
