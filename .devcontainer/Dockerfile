# Copyright (c) 2021 Tailscale Inc & AUTHORS All rights reserved.
# Use of this source code is governed by a BSD-style
# license that can be found in the LICENSE file.

FROM mcr.microsoft.com/vscode/devcontainers/universal:linux as builder
ARG KUBECTL_VERSION

## Install Kubectl
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl && \
    chmod +x ./kubectl && \
    sudo mv ./kubectl /usr/local/bin/kubectl && \
    mkdir ~/.kube

## Install Helm
RUN curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 && \
    chmod 700 get_helm.sh && \
    ./get_helm.sh

## Install Kustomize
RUN curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash

## Install dependencies
RUN sudo apt update && \
    sudo apt install fzf

## Install easy ctx/ns switcher
RUN git clone https://github.com/blendle/kns.git && \
    cd kns/bin && \
    chmod +x kns && sudo mv kns /usr/local/bin && \
    chmod +x ktx && sudo mv ktx /usr/local/bin

## Install Krew
RUN set -x; cd "$(mktemp -d)" && \
    OS="$(uname | tr '[:upper:]' '[:lower:]')" && \
    ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" && \
    KREW="krew-${OS}_${ARCH}" && \
    curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" && \
    tar zxvf "${KREW}.tar.gz" && \
    ./"${KREW}" install krew && \
    echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> /home/gitpod/.bashrc

## Install Krew main plugins
RUN export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH" && \
    kubectl krew install neat && \
    kubectl krew install access-matrix && \
    kubectl krew install advise-psp && \
    kubectl krew install cert-manager && \
    kubectl krew install ca-cert && \
    kubectl krew install get-all && \
    kubectl krew install ingress-nginx

# Add aliases
RUN echo 'alias k="kubectl"' >> /home/gitpod/.bashrc

USER root

# Magic DNS in a container where /etc/resolv.conf is a bind mount needed
# extra support, currently on a development branch.
WORKDIR /go/src/tailscale
COPY . ./
RUN git clone https://github.com/tailscale/tailscale.git && cd tailscale && \
    go mod download && \
    go install -mod=readonly ./cmd/tailscaled ./cmd/tailscale
COPY . ./

FROM mcr.microsoft.com/vscode/devcontainers/universal:linux
USER root

RUN apt-get update && apt-get install -y curl gpg dnsutils
COPY tailscaled /etc/init.d
COPY --from=builder /go/bin/tailscaled /usr/sbin/tailscaled
COPY --from=builder /go/bin/tailscale /usr/bin/tailscale

RUN mkdir -p /var/run/tailscale /var/cache/tailscale /var/lib/tailscale
