FROM debian:stable-slim

ARG DEBIAN_FRONTEND=noninteractive

# Default
RUN apt update && apt upgrade -y

# Dependencies
RUN apt install -y curl ca-certificates xz-utils libatomic1 fd-find ripgrep
RUN ln -s /usr/bin/fdfind /usr/bin/fd

# Download and extract nodejs
RUN <<EOF
curl --fail --show-error --silent --location --output /tmp/node-v26.13.1-linux-x64.tar.xz \
    https://nodejs.org/dist/v26.13.1/node-v26.13.1-linux-x64.tar.xz
tar -xf /tmp/node-v26.13.1-linux-x64.tar.xz -C /opt/
rm /tmp/node-v26.13.1-linux-x64.tar.xz
EOF

ENV PATH="/opt/node-v26.13.1-linux-x64/bin:${PATH}"

RUN npm install -g --ignore-scripts --no-fund --loglevel=error --no-audit --progress=false "@earendil-works/pi-coding-agent"

CMD ["pi"]
