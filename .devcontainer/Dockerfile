FROM debian:bookworm-slim AS base

ENV LANG=C.UTF-8 \
    USERNAME=vscode \
    USER_UID=1000 \
    USER_GID=1000 \
    DEBIAN_FRONTEND=noninteractive

ARG GHC_VERSION=9.4.8
ARG STACK_VERSION=recommended
ARG STACK_RESOLVER=lts-21.25
ARG CABAL_VERSION=recommended
ARG HLS_VERSION=recommended
ARG LLVM_VERSION=17

ENV GHC_VERSION=${GHC_VERSION} \
    STACK_VERSION=${STACK_VERSION} \
    STACK_RESOLVER=${STACK_RESOLVER} \
    CABAL_VERSION=${CABAL_VERSION} \
    HLS_VERSION=${HLS_VERSION} \
    LLVM_VERSION=${LLVM_VERSION}

# Install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        software-properties-common \
        wget \
        apt-utils \
        bash \
        build-essential \
        ca-certificates \
        curl \
        gcc \
        git \
        gnupg \
        libffi8 \
        libffi-dev \
        libgmp-dev \
        libgmp10 \
        libicu-dev \
        libncurses5 \
        libncurses-dev \
        libnuma-dev \
        libtinfo5 \
        lsb-release \
        make \
        openssh-client \
        procps \
        sudo \
        xz-utils \
        z3 \
        zlib1g-dev && \
    VERSION_CODENAME=$(grep VERSION_CODENAME /etc/os-release | cut -d'=' -f2) && \
    add-apt-repository -y -s -n "deb http://apt.llvm.org/${VERSION_CODENAME}/ llvm-toolchain-${VERSION_CODENAME}-${LLVM_VERSION} main" && \
    add-apt-repository -y -s -n "deb http://apt.llvm.org/${VERSION_CODENAME}/ llvm-toolchain-${VERSION_CODENAME}-${LLVM_VERSION} main" && \
    wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        clang-${LLVM_VERSION} \
        lldb-${LLVM_VERSION} \
        lld-${LLVM_VERSION} \
        clangd-${LLVM_VERSION} && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create user
RUN groupadd --gid ${USER_GID} ${USERNAME} && \
    useradd -ms /bin/bash -K MAIL_DIR=/dev/null --uid ${USER_UID} --gid ${USER_GID} -m ${USERNAME} && \
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} && \
    chmod 0440 /etc/sudoers.d/${USERNAME}

USER ${USERNAME}
WORKDIR /home/${USERNAME}
ENV PATH="/home/${USERNAME}/.local/bin:/home/${USERNAME}/.cabal/bin:/home/${USERNAME}/.ghcup/bin:$PATH"

RUN echo "export PATH=${PATH}" >> ~/.profile

ENV BOOTSTRAP_HASKELL_NONINTERACTIVE=yes \
    BOOTSTRAP_HASKELL_NO_UPGRADE=yes

# Install Haskell tooling
FROM base AS tooling

RUN curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh && \
    ghcup install ghc ${GHC_VERSION} --set && \
    ghcup install cabal ${CABAL_VERSION} --set && \
    cabal update && \
    ghcup install stack ${STACK_VERSION} --set && \
    ((stack ghc -- --version 2>/dev/null) || true) && \
    stack config --system-ghc set system-ghc true --global && \
    stack config --system-ghc set install-ghc false --global && \
    stack config --system-ghc set resolver ${STACK_RESOLVER} && \
    printf "ghc-options:\n  \"\$everything\": -haddock\n" >> ~/.stack/config.yaml && \
    ghcup install hls ${HLS_VERSION} --set 

# Configure cabal
RUN cabal user-config update -f && \
    sed -i 's/-- ghc-options:/ghc-options: -haddock/g' ~/.cabal/config

ENV DEBIAN_FRONTEND=dialog
ENTRYPOINT ["/bin/bash"]
