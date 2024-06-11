# base image
FROM alpine:3.18

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

# your custom setup goes here
RUN wget -O caddy "https://caddyserver.com/api/download?os=linux&arch=arm64&p=github.com%2Fcaddy-dns%2Fnamecheap"
RUN mv caddy /usr/bin/caddy
RUN chmod +x /usr/bin/caddy
ENV XDG_DATA_HOME=/var/lib

COPY Caddyfile /etc/caddy/Caddyfile
COPY ipfsLink /app/ipfsLink
COPY ipfsLinkConfig.json /app/ipfsLinkConfig.json
COPY acme-id /app/acme-id
COPY ipfs /app/ipfs

# entry point
ENTRYPOINT [ "/app/setup.sh" ]