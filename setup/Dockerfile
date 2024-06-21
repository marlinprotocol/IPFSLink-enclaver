# ipfsLink build from nodejs
FROM node:22-alpine3.18 as node-builder

WORKDIR /build

RUN apk add git alpine-sdk python3 linux-headers
RUN git clone -b upgrade/sns-v2 https://github.com/marlinprotocol/ipfsLink.git
RUN cd ipfsLink && npm i -f && npm run build

# acme-id build from rust
FROM rust:alpine3.18 as rust-builder

WORKDIR /build

RUN apk add git alpine-sdk linux-headers
RUN git clone https://github.com/marlinprotocol/oyster-caddy-account-uri.git
RUN cd oyster-caddy-account-uri/signer && cargo build --release

# kubo build
FROM golang:alpine3.18 as go-builder

WORKDIR /build

RUN apk add alpine-sdk bash binutils-gold
RUN git clone https://github.com/ipfs/kubo.git
RUN cd kubo && make build

# base image
FROM node:22-alpine3.18 as enclave-builder

# install dependency tools
RUN apk add --no-cache net-tools iptables iproute2 wget ipset

# working directory
WORKDIR /app

# supervisord to manage programs
RUN wget -O supervisord http://public.artifacts.marlin.pro/projects/enclaves/supervisord_master_linux_arm64
RUN chmod +x supervisord

# proxy component inside the enclave to enable outgoing connections
RUN wget -O ip-to-vsock-raw-outgoing http://public.artifacts.marlin.pro/projects/enclaves/ip-to-vsock-raw-outgoing_v1.0.0_linux_arm64
RUN chmod +x ip-to-vsock-raw-outgoing

# proxy component inside the enclave to enable incoming connections
RUN wget -O vsock-to-ip-raw-incoming http://public.artifacts.marlin.pro/projects/enclaves/vsock-to-ip-raw-incoming_v1.0.0_linux_arm64
RUN chmod +x vsock-to-ip-raw-incoming

# key generator to generate static keys
RUN wget -O keygen http://public.artifacts.marlin.pro/projects/enclaves/keygen_v1.0.0_linux_arm64
RUN chmod +x keygen

# attestation server inside the enclave that generates attestations
RUN wget -O attestation-server http://public.artifacts.marlin.pro/projects/enclaves/attestation-server_v1.0.0_linux_arm64
RUN chmod +x attestation-server

# dnsproxy to provide DNS services inside the enclave
RUN wget -O dnsproxy http://public.artifacts.marlin.pro/projects/enclaves/dnsproxy_v0.46.5_linux_arm64
RUN chmod +x dnsproxy

# vsock http querier to query setup params
RUN wget -O vet http://public.artifacts.marlin.pro/projects/enclaves/vet_v1.0.0_linux_arm64
RUN chmod +x vet

# supervisord config
COPY supervisord.conf /etc/supervisord.conf

# setup.sh script that will act as entrypoint
COPY setup.sh ./
RUN chmod +x setup.sh

# setup caddy with namecheap plugin
RUN wget -O caddy "https://caddyserver.com/api/download?os=linux&arch=arm64&p=github.com%2Fcaddy-dns%2Fnamecheap"
RUN mv caddy /usr/bin/caddy
RUN chmod +x /usr/bin/caddy
ENV XDG_DATA_HOME=/var/lib

# copy from builders
COPY --from=node-builder --chown=10005:10005 /build/ipfsLink/dist/ /app/dist/
COPY --from=rust-builder --chown=10005:10005 /build/oyster-caddy-account-uri/signer/target/release/signer /app/acme-id
COPY --from=go-builder --chown=10005:10005 /build/kubo/cmd/ipfs/ipfs /app/ipfs

ARG NAMECHEAP_USER
ARG NAMECHEAP_API_KEY

ENV NAMECHEAP_USER=${NAMECHEAP_USER}
ENV NAMECHEAP_API_KEY=${NAMECHEAP_API_KEY}

# copy config
COPY Caddyfile /etc/caddy/Caddyfile
COPY ipfsLinkConfig.json /app/ipfsLinkConfig.json

# entry point
ENTRYPOINT [ "/app/setup.sh" ]