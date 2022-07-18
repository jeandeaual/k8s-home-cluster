# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.217.1/containers/python-3/.devcontainer/base.Dockerfile

ARG VARIANT="3.10-bullseye"
FROM mcr.microsoft.com/vscode/devcontainers/python:0-${VARIANT}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# [Choice] Node.js version: none, lts/*, 18, 16, 14, 12, 10
ARG NODE_VERSION="none"
RUN if [[ "${NODE_VERSION}" != "none" ]]; then su vscode -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"; fi

ARG KUBECTX_VERSION=0.9.4
ARG FLUX_VERSION=0.31.3
ARG YQ_VERSION=4.25.3
ARG K9S_VERSION=0.25.21
ARG SOPS_VERSION=3.7.3
ARG GO_TASK_VERSION=3.14.0

# hadolint ignore=DL3008
# Docker
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list \
    # Kubernetes
    && curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list \
    # Helm
    && curl -fsSL https://baltocdn.com/helm/signing.asc | gpg --dearmor -o /usr/share/keyrings/helm.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" > /etc/apt/sources.list.d/helm-stable-debian.list \
    # Enable Backports for age
    && echo "deb http://deb.debian.org/debian bullseye-backports main" > /etc/apt/sources.list.d/backports.list \
    && apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
        docker-ce \
        docker-ce-cli \
        containerd.io \
        docker-compose-plugin \
        kubectl=1.24.3-00 \
        helm=3.9.1-1 \
        age \
        ipcalc \
        jq \
        direnv \
        sshpass \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    # kubectx
    && curl -fsSL "https://github.com/ahmetb/kubectx/releases/download/v${KUBECTX_VERSION}/kubectx_v${KUBECTX_VERSION}_linux_arm64.tar.gz" | tar -xz -C /usr/local/bin kubectx \
    # kubens
    && curl -fsSL "https://github.com/ahmetb/kubectx/releases/download/v${KUBECTX_VERSION}/kubens_v${KUBECTX_VERSION}_linux_arm64.tar.gz" | tar -xz -C /usr/local/bin kubens \
    # flux
    && curl -fsSL "https://github.com/fluxcd/flux2/releases/download/v${FLUX_VERSION}/flux_${FLUX_VERSION}_linux_amd64.tar.gz" | tar -xz -C /usr/local/bin flux \
    # yq
    && curl -fsSL "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_amd64.tar.gz" | tar -xz -C /usr/local/bin ./yq_linux_amd64 \
    && mv /usr/local/bin/yq{_linux_amd64,} \
    # k9s
    && curl -fsSL "https://github.com/derailed/k9s/releases/download/v${K9S_VERSION}/k9s_Linux_arm64.tar.gz" | tar -xz -C /usr/local/bin k9s \
    # sops
    && curl -fsSLo /tmp/sops.deb "https://github.com/mozilla/sops/releases/download/v${SOPS_VERSION}/sops_${SOPS_VERSION}_amd64.deb" \
    && dpkg -i /tmp/sops.deb \
    # go-task
    && curl -fsSLo /tmp/task.deb "https://github.com/go-task/task/releases/download/v${GO_TASK_VERSION}/task_linux_amd64.deb" \
    && dpkg -i /tmp/task.deb \
    && rm -f /tmp/*.deb

USER vscode

ARG POETRY_VERSION="1.2.0b2"

RUN curl -sSL https://install.python-poetry.org | POETRY_VERSION="${POETRY_VERSION}" python -

# Install prettier
RUN npm install -g prettier@2.7.1
