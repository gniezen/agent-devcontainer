FROM node:20-bookworm

RUN npm install -g \
      @anthropic-ai/claude-code \
      @earendil-works/pi-coding-agent

# Create config dirs owned by node so volume mounts inherit ownership
RUN mkdir -p /home/node/.claude /home/node/.pi/agent && \
      chown -R node:node /home/node/.claude /home/node/.pi

# Install packages the firewall script needs
RUN apt-get update && apt-get install -y --no-install-recommends \
      iptables ipset iproute2 dnsutils aggregate jq curl sudo \
 && rm -rf /var/lib/apt/lists/*

# Copy the firewall script in
COPY init-firewall.sh /usr/local/bin/init-firewall.sh
RUN chmod +x /usr/local/bin/init-firewall.sh

# Grant the non-root user passwordless sudo *only* for this script
RUN echo "node ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" \
      > /etc/sudoers.d/node-firewall && \
    chmod 0440 /etc/sudoers.d/node-firewall

USER node
WORKDIR /workspace
