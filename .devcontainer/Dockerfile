FROM node:22-bookworm

# Root-only setup first
RUN apt-get update && apt-get install -y --no-install-recommends \
      iptables ipset iproute2 dnsutils aggregate jq curl sudo \
 && rm -rf /var/lib/apt/lists/*

COPY init-firewall.sh /usr/local/bin/init-firewall.sh
RUN chmod +x /usr/local/bin/init-firewall.sh

RUN echo "node ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" \
      > /etc/sudoers.d/node-firewall && \
    chmod 0440 /etc/sudoers.d/node-firewall

# Config dirs owned by node so volume mounts inherit ownership
RUN mkdir -p /home/node/.claude /home/node/.pi/agent && \
      chown -R node:node /home/node/.claude /home/node/.pi

# node-owned npm global prefix so Claude can write its own updates
ENV NPM_CONFIG_PREFIX=/home/node/.npm-global
ENV PATH=/home/node/.npm-global/bin:$PATH
RUN mkdir -p /home/node/.npm-global && chown -R node:node /home/node/.npm-global

USER node
RUN npm install -g \
      @anthropic-ai/claude-code \
      @earendil-works/pi-coding-agent

WORKDIR /workspace
