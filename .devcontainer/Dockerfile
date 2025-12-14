FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

ARG GO_VERSION=1.23.4
ARG HUGO_VERSION=0.152.2
ARG HTMLTEST_VERSION=0.17.0

# Base utilities (no Node)
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    git \
    tar \
    xz-utils \
    gzip \
  && rm -rf /var/lib/apt/lists/*

# Go (pinned)
RUN curl -fsSL "https://dl.google.com/go/go${GO_VERSION}.linux-amd64.tar.gz" -o /tmp/go.tgz \
  && rm -rf /usr/local/go \
  && tar -C /usr/local -xzf /tmp/go.tgz \
  && rm -f /tmp/go.tgz

ENV PATH="/usr/local/go/bin:${PATH}"

# Hugo extended (pinned + checksum verify)
RUN curl -fsSL "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_checksums.txt" -o /tmp/hugo_checksums.txt \
  && curl -fsSL "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz" -o "/tmp/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz" \
  && grep "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz" /tmp/hugo_checksums.txt > /tmp/hugo_sha256.txt \
  && (cd /tmp && sha256sum -c /tmp/hugo_sha256.txt) \
  && tar -C /usr/local/bin -xzf "/tmp/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz" hugo \
  && rm -f /tmp/hugo_checksums.txt "/tmp/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz" /tmp/hugo_sha256.txt

# htmltest (pinned)
RUN curl -fsSL "https://github.com/wjdp/htmltest/releases/download/v${HTMLTEST_VERSION}/htmltest_${HTMLTEST_VERSION}_linux_amd64.tar.gz" -o /tmp/htmltest.tgz \
  && tar -C /usr/local/bin -xzf /tmp/htmltest.tgz htmltest \
  && rm -f /tmp/htmltest.tgz

# Default workdir
WORKDIR /workspaces/heirbaut.nl
