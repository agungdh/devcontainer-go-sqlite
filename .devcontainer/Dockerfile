# Debian 13 (trixie) - Go devcontainer
FROM debian:13

ARG DEBIAN_FRONTEND=noninteractive
ARG USERNAME=agungdh
ARG USER_UID=1000
ARG USER_GID=1000

# ---------------------------------------------------------------------------
# Base packages (Go dari apt; versi toolchain terbaru di-download otomatis
# lewat GOTOOLCHAIN=auto saat go.mod butuh versi lebih baru)
# ---------------------------------------------------------------------------
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        wget \
        git \
        openssh-client \
        gnupg \
        sudo \
        make \
        jq \
        unzip \
        zip \
        xz-utils \
        tar \
        less \
        vim \
        nano \
        procps \
        htop \
        tree \
        ripgrep \
        file \
        locales \
        golang-go \
        sqlite3 \
    && rm -rf /var/lib/apt/lists/*

# Locale (UTF-8)
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && locale-gen
ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

# ---------------------------------------------------------------------------
# Non-root user with passwordless sudo
# ---------------------------------------------------------------------------
RUN groupadd --gid ${USER_GID} ${USERNAME} \
    && useradd --uid ${USER_UID} --gid ${USER_GID} --create-home --shell /bin/bash ${USERNAME} \
    && echo "${USERNAME} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME} \
    && mkdir -p /workspaces \
    && chown ${USER_UID}:${USER_GID} /workspaces

# ---------------------------------------------------------------------------
# Go environment
# ---------------------------------------------------------------------------
ENV GOPATH=/home/${USERNAME}/go \
    GOTOOLCHAIN=auto \
    PATH=/home/${USERNAME}/go/bin:/usr/local/go/bin:$PATH

# ---------------------------------------------------------------------------
# Docker CLI only (daemon di host, container pakai /var/run/docker.sock)
# Sumber repo resmi: https://docs.docker.com/engine/install/debian/
# ---------------------------------------------------------------------------
RUN install -m 0755 -d /etc/apt/keyrings \
    && curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
        | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        docker-ce-cli \
        docker-buildx-plugin \
        docker-compose-plugin \
    && rm -rf /var/lib/apt/lists/*

# Samakan GID grup docker dengan socket host saat container start
RUN printf '%s\n' \
        '#!/bin/sh' \
        'set -e' \
        'if [ -S /var/run/docker.sock ]; then' \
        '    gid=$(stat -c "%g" /var/run/docker.sock)' \
        '    if getent group docker >/dev/null 2>&1; then' \
        '        current=$(getent group docker | cut -d: -f3)' \
        '        [ "$current" = "$gid" ] || sudo groupmod --gid "$gid" docker' \
        '    else' \
        '        sudo groupadd --gid "$gid" docker' \
        '    fi' \
        '    sudo usermod -aG docker '"${USERNAME}" \
        'fi' \
        'exec "$@"' \
        > /usr/local/share/docker-init.sh \
    && chmod +x /usr/local/share/docker-init.sh

# ---------------------------------------------------------------------------
# opencode
# ---------------------------------------------------------------------------
USER ${USERNAME}
RUN curl -fsSL https://opencode.ai/v2/install | bash
ENV PATH=/home/${USERNAME}/.opencode/bin:$PATH

# ---------------------------------------------------------------------------
# Node (nvm + Node.js 24)
# ---------------------------------------------------------------------------
ENV NVM_DIR=/home/${USERNAME}/.nvm
RUN curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.41.0/install.sh | bash \
    && bash -c '. "${NVM_DIR}/nvm.sh" && nvm install 24 && nvm alias default 24'

# Login shell (/etc/profile) me-reset PATH, jadi daftarkan ulang via profile.d
USER root
RUN PROFILE_PATH="/home/${USERNAME}/.opencode/bin:/home/${USERNAME}/go/bin:/usr/local/go/bin" \
    && echo "export PATH=\"${PROFILE_PATH}:\$PATH\"" \
        | tee /etc/profile.d/devcontainer-path.sh > /dev/null \
    && echo "export NVM_DIR=\"${NVM_DIR}\"; [ -s \"\$NVM_DIR/nvm.sh\" ] && . \"\$NVM_DIR/nvm.sh\"" \
        | tee /etc/profile.d/nvm.sh > /dev/null

WORKDIR /workspaces

ENTRYPOINT ["/usr/local/share/docker-init.sh"]
CMD ["sleep", "infinity"]
