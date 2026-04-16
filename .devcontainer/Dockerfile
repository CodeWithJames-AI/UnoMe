FROM mcr.microsoft.com/devcontainers/typescript-node:22

# Install Rust
ARG CARGO_HOME="/usr/local/cargo"
ARG RUSTUP_HOME="/usr/local/rustup"
ENV CARGO_HOME=${CARGO_HOME} RUSTUP_HOME=${RUSTUP_HOME} PATH=${CARGO_HOME}/bin:${PATH}
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --profile minimal --component rust-analyzer,rust-src,rustfmt,clippy

# Install pnpm, Claude Code
RUN npm install -g pnpm@10.20.0 @anthropic-ai/claude-code

# Install GitHub CLI
RUN ARCH=$(dpkg --print-architecture) && echo "deb [arch=$ARCH signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" > /etc/apt/sources.list.d/github-cli.list && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg && chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg && apt-get update -qq && apt-get install -y -qq gh

# Copy source into container's own filesystem (overlayfs — chmod works)
COPY . /workspaces/UnoMe

# Install deps inside container
RUN cd /workspaces/UnoMe/buildingai && pnpm install || true

# Git safe directory
RUN git config --global --add safe.directory /workspaces/UnoMe

WORKDIR /workspaces/UnoMe
