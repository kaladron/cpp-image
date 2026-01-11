ARG DEBIAN_VERSION=testing-slim
FROM debian:${DEBIAN_VERSION}

USER root

# Create vscode user similar to base image
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN if id -u $USER_UID &>/dev/null; then \
        existing_user=$(id -un $USER_UID); \
        echo "Removing existing user $existing_user with UID $USER_UID"; \
        userdel -r $existing_user || true; \
    fi \
    && if getent group $USER_GID &>/dev/null; then \
        existing_group=$(getent group $USER_GID | cut -d: -f1); \
        echo "Removing existing group $existing_group with GID $USER_GID"; \
        groupdel $existing_group || true; \
    fi \
    && groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # Add sudo support for the non-root user
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Install common utilities like git, oh-my-zsh, etc
ARG CMAKE_URL="https://apt.kitware.com/debian/"
ARG CMAKE_CODENAME_FALLBACK=bookworm
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
        git \
        openssh-client \
        less \
        iproute2 \
        procps \
        lsb-release \
        ca-certificates \
        apt-transport-https \
        gnupg2 \
        curl \
        wget \
        unzip \
        nano \
        jq \
        zsh \
        build-essential \
        gdb \
        clang \
        pkg-config \
        libssl-dev \
        openssl \
    && mkdir -p /etc/apt/keyrings \
    && wget -qO /tmp/kitware-archive-keyring.asc https://apt.kitware.com/keys/kitware-archive-latest.asc \
    && gpg --batch --show-keys --with-colons /tmp/kitware-archive-keyring.asc | grep -q "fpr:::::::::4DBEBE3EEC96E7B8C6EC5BE99E92FDC6C5B9BA75:" \
    && gpg --dearmor /tmp/kitware-archive-keyring.asc -o /etc/apt/keyrings/kitware-archive.gpg \
    && rm -f /tmp/kitware-archive-keyring.asc \
    && . /etc/os-release \
    && cmake_codename="${VERSION_CODENAME:-${CMAKE_CODENAME_FALLBACK}}" \
    && if [ "${VERSION:-}" = "testing" ] || [ "${VERSION:-}" = "unstable" ] || [ "$cmake_codename" = "forky" ] || [ "$cmake_codename" = "trixie" ]; then \
         cmake_codename="${CMAKE_CODENAME_FALLBACK}"; \
       fi \
    && echo "deb [signed-by=/etc/apt/keyrings/kitware-archive.gpg] ${CMAKE_URL} ${cmake_codename} main" \
        > /etc/apt/sources.list.d/kitware.list \
    && apt-get update -qq \
    && apt-get install -y --no-install-recommends cmake cmake-curses-gui \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Install LLVM apt repository and full clang 22 toolchain plus ninja-build
ARG LLVM_VERSION_CODENAME=
RUN set -eux \
    && . /etc/os-release \
    && llvm_version=22 llvm_suffix="" \
    && VERSION="${VERSION:-}" \
    && repo_codename="${LLVM_VERSION_CODENAME:-${VERSION_CODENAME:-bookworm}}" \
    && repo_suite="llvm-toolchain-${repo_codename}" \
    && if [ "$ID" = "debian" ] && { [ "${VERSION}" = "unstable" ] || [ "${VERSION}" = "testing" ] || [ "$repo_codename" = "forky" ] || [ "$repo_codename" = "trixie" ]; }; then \
         repo_codename=unstable; repo_suite=llvm-toolchain; \
       fi \
    && curl -fsSL https://apt.llvm.org/llvm-snapshot.gpg.key | gpg --dearmor -o /usr/share/keyrings/llvm-snapshot.gpg \
    && printf "Types: deb\nURIs: http://apt.llvm.org/%s/\nSuites: %s\nComponents: main\nArchitectures: amd64\nSigned-By: /usr/share/keyrings/llvm-snapshot.gpg\n" "${repo_codename}" "${repo_suite}" \
        | tee /etc/apt/sources.list.d/llvm-toolchain.sources >/dev/null \
    && apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install -y --no-install-recommends \
        clang-22 \
        clangd-22 \
        clang-format-22 \
        clang-tidy-22 \
        clang-tools-22 \
        llvm-22-dev \
        llvm-22-tools \
        lld-22 \
        lldb-22 \
        libclang-common-22-dev \
        libclang-22-dev \
        libclang-cpp22-dev \
        libclang-rt-22-dev \
        liblldb-22-dev \
        libpolly-22-dev \
        libfuzzer-22-dev \
        libomp-22-dev \
        libc++-22-dev \
        libc++abi-22-dev \
        libunwind-22-dev \
        ninja-build \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Install oh-my-zsh for the vscode user
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended --user $USERNAME \
    && chsh -s /usr/bin/zsh $USERNAME

# Update CA certificates to fix SSL issues
RUN update-ca-certificates

USER $USERNAME
