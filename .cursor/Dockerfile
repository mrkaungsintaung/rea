FROM node:24.22.1-bookworm AS node_runtime
FROM ubuntu:24.04
COPY --from=node_runtime /usr/local/ /usr/local/
ENV PATH="/usr/local/bin:${PATH}"
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates curl git procps python3 make g++ unzip xz-utils sudo && rm -rf /var/lib/apt/lists/*
RUN npm install --global npm@11.16.0
