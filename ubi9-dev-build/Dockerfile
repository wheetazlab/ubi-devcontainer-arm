# This is my ghcr.io image.  You can create your own free quay.io account or ghcr.io account and push your image there.
# I use Podman desktop to push the image to ghcr.io.  You can push to ghcr.io the Podman desktop.
# you can use the podman extension in VS Code to build and push the image to quay.io or ghcr.io or run the following command in the terminal.
# change directory into ubi9-dev-build
# "podman build -t ghcr.io/wheetazlab/ubi-devcontainer-arm:latest ."  (but this will need to be your registry)

# syntax=docker/dockerfile:1.3-labs

# updateBaseImages.sh can't operate on SHA-based tags as they're not date-based or semver-sequential, and therefore cannot be ordered
FROM quay.io/devfile/base-developer-image:ubi9-latest

LABEL org.opencontainers.image.source=https://github.com/wheetazlab/ubi-devcontainer-arm:latest
LABEL name="wheetazlab/ubi-devcontainer-arm:latest"
LABEL version="latest"


#labels for container catalog
LABEL io.openshift.expose-services=""

USER 0

# $PROFILE_EXT contains all additions made to the bash environment
ENV PROFILE_EXT=/etc/profile.d/udi_environment.sh
RUN touch ${PROFILE_EXT} & chown 10001 ${PROFILE_EXT}

# Disable subscription-manager plugin
RUN dnf remove subscription-manager -y

USER 10001

# We install everything to /home/tooling/ as /home/user/ may get overriden, see github.com/eclipse/che/issues/22412
ENV HOME=/home/tooling

# Java
# RUN curl -fsSL "https://get.sdkman.io/?rcupdate=false" | bash \
#     && bash -c ". /home/tooling/.sdkman/bin/sdkman-init.sh \
#              && sed -i "s/sdkman_auto_answer=false/sdkman_auto_answer=true/g" /home/tooling/.sdkman/etc/config \
# 	     && sed -i "s/sdkman_auto_env=false/sdkman_auto_env=true/g" /home/tooling/.sdkman/etc/config \
#              && sdk install java 8.0.432-tem \
#              && sdk install java 11.0.25-tem \
#              && sdk install java 17.0.13-tem \
#              && sdk install java 21.0.5-tem \
#              && sdk install java 23.1.5.r21-mandrel \
#              && sdk default java 17.0.13-tem \
#              && sdk install gradle \
#              && sdk install maven \
#              && sdk install jbang \
#              && sdk flush archives \
#              && sdk flush temp" \
#          && chgrp -R 0 /home/tooling && chmod -R g=u /home/tooling

# # sdk home java <version>
# ENV JAVA_HOME_8=/home/tooling/.sdkman/candidates/java/8.0.432-tem
# ENV JAVA_HOME_11=/home/tooling/.sdkman/candidates/java/11.0.25-tem
# ENV JAVA_HOME_17=/home/tooling/.sdkman/candidates/java/17.0.13-tem
# ENV JAVA_HOME_21=/home/tooling/.sdkman/candidates/java/21.0.5-tem

# Java-related environment variables are described and set by ${PROFILE_EXT}, which will be loaded by ~/.bashrc
# To make Java working for dash and other shells, it needs to initialize them in the Dockerfile.
# ENV SDKMAN_CANDIDATES_API="https://api.sdkman.io/2"
# ENV SDKMAN_CANDIDATES_DIR="/home/tooling/.sdkman/candidates"
# ENV SDKMAN_DIR="/home/tooling/.sdkman"
# ENV SDKMAN_PLATFORM="linuxx64"
# ENV SDKMAN_VERSION="5.18.2"

# ENV GRADLE_HOME="/home/tooling/.sdkman/candidates/gradle/current"
# ENV JAVA_HOME="/home/tooling/.sdkman/candidates/java/current"
# ENV MAVEN_HOME="/home/tooling/.sdkman/candidates/maven/current"

# ENV GRAALVM_HOME=/home/tooling/.sdkman/candidates/java/23.1.5.r21-mandrel

# ENV PATH="/home/tooling/.krew/bin:$PATH"
# ENV PATH="/home/tooling/.sdkman/candidates/maven/current/bin:$PATH"
# ENV PATH="/home/tooling/.sdkman/candidates/java/current/bin:$PATH"
# ENV PATH="/home/tooling/.sdkman/candidates/gradle/current/bin:$PATH"
# ENV PATH="/home/tooling/.local/share/coursier/bin:$PATH"

# NodeJS
# RUN mkdir -p /home/tooling/.nvm/
# ENV NVM_DIR="/home/tooling/.nvm"
# ENV NODEJS_20_VERSION=20.18.1
# ENV NODEJS_18_VERSION=18.20.5
# ENV NODEJS_DEFAULT_VERSION=${NODEJS_18_VERSION}
# RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | PROFILE=/dev/null bash
# RUN echo 'export NVM_DIR="$HOME/.nvm"' >> ${PROFILE_EXT} \
#     && echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ${PROFILE_EXT}
# RUN source /home/user/.bashrc && \
#     nvm install v${NODEJS_20_VERSION} && \
#     nvm install v${NODEJS_18_VERSION} && \
#     nvm alias default v${NODEJS_DEFAULT_VERSION} && nvm use v${NODEJS_DEFAULT_VERSION} && \
#     npm install --global yarn@v1.22.22 &&\
#     chgrp -R 0 /home/tooling && chmod -R g=u /home/tooling
# ENV PATH=$NVM_DIR/versions/node/v${NODEJS_DEFAULT_VERSION}/bin:$PATH
# ENV NODEJS_HOME_20=$NVM_DIR/versions/node/v${NODEJS_20_VERSION}
# ENV NODEJS_HOME_18=$NVM_DIR/versions/node/v${NODEJS_18_VERSION}

# kube
ENV KUBECONFIG=/home/user/.kube/config

USER 0

# Don't include container-selinux and remove
# directories used by yum that are just taking
# up space.
RUN sudo yum -y module install container-tools; dnf -y update; rpm --restore --quiet shadow-utils; \
    dnf -y install crun podman netavark fuse-overlayfs /etc/containers/storage.conf --exclude container-selinux; \
    rm -rf /var/cache/* /var/log/dnf* /var/log/yum.*

RUN useradd podman; \
    echo podman:10000:5000 > /etc/subuid; \
    echo podman:10000:5000 > /etc/subgid;

# Setup internal Podman to pass subscriptions down from host to internal container
# https://issues.redhat.com/browse/RHEL-40706
RUN printf '/run/secrets/etc-pki-entitlement:/run/secrets/etc-pki-entitlement\n/run/secrets/rhsm:/run/secrets/rhsm\n' > /etc/containers/mounts.conf

VOLUME /var/lib/containers
RUN mkdir -p /home/podman/.local/share/containers
RUN mkdir -p /home/podman/.config
RUN chown podman:podman -R /home/podman
VOLUME /home/podman/.local/share/containers

# https://raw.githubusercontent.com/containers/libpod/master/contrib/podmanimage/stable/containers.conf
#ADD containers.conf /etc/containers/containers.conf
# https://raw.githubusercontent.com/containers/libpod/master/contrib/podmanimage/stable/podman-containers.conf
#ADD podman-containers.conf /home/podman/.config/containers/containers.conf

# chmod containers.conf and adjust storage.conf to enable Fuse storage.
#RUN chmod 644 /etc/containers/containers.conf; sed -i -e 's|^#mount_program|mount_program|g' -e '/additionalimage.*/a "/var/lib/shared",' -e 's|^mountopt[[:space:]]*=.*$|mountopt = "nodev,fsync=0"|g' /etc/containers/storage.conf
#RUN mkdir -p /var/lib/shared/overlay-images /var/lib/shared/overlay-layers /var/lib/shared/vfs-images /var/lib/shared/vfs-layers; touch /var/lib/shared/overlay-images/images.lock; touch /var/lib/shared/overlay-layers/layers.lock; touch /var/lib/shared/vfs-images/images.lock; touch /var/lib/shared/vfs-layers/layers.lock

ENV _CONTAINERS_USERNS_CONFIGURED=""

# Getting podman to run
RUN mv /usr/bin/podman.orig /usr/bin/podman

# Required packages for AWT
# RUN dnf install -y libXext libXrender libXtst libXi

# Lombok
# ENV LOMBOK_VERSION=1.18.18
# RUN wget -O /usr/local/lib/lombok.jar https://projectlombok.org/downloads/lombok-${LOMBOK_VERSION}.jar

# # Scala
# RUN curl -fLo cs https://git.io/coursier-cli && \
#     chmod +x cs && \
#     mv cs /usr/local/bin/
# RUN curl -fLo sbt https://raw.githubusercontent.com/dwijnand/sbt-extras/master/sbt && \
#     chmod +x sbt && \
#     mv sbt /usr/local/bin/
# RUN curl -fLo mill https://raw.githubusercontent.com/lefou/millw/main/millw && \
#     chmod +x mill && \
#     mv mill /usr/local/bin/

# C/CPP
RUN dnf -y install llvm-toolset gcc gcc-c++ clang clang-libs clang-tools-extra gdb

# Go 1.18+    - installed to /usr/bin/go
# gopls 0.10+ - installed to /home/tooling/go/bin/gopls and /home/tooling/go/pkg/mod/
RUN dnf install -y go-toolset && \
    GO111MODULE=on go install -v golang.org/x/tools/gopls@latest && \
    chgrp -R 0 /home/tooling && chmod -R g=u /home/tooling
ENV GOBIN="/home/tooling/go/bin/"
ENV PATH="$GOBIN:$PATH"

# Python
RUN dnf -y install python3.11 python3.11-devel python3.11-setuptools python3.11-pip nss_wrapper

RUN cd /usr/bin \
    && if [ ! -L python ]; then ln -s python3.11 python; fi \
    && if [ ! -L pydoc ]; then ln -s pydoc3.11 pydoc; fi \
    && if [ ! -L python-config ]; then ln -s python3.11-config python-config; fi \
    && if [ ! -L pip ]; then ln -s pip-3.11 pip; fi

RUN pip install pylint yq

# PHP
# ENV PHP_VERSION=8.2
# RUN dnf -y module enable php:$PHP_VERSION && \
#     dnf install -y --setopt=tsflags=nodocs php php-mysqlnd php-pgsql php-bcmath \
#                 php-gd php-intl php-json php-ldap php-mbstring php-pdo \
#                 php-pear php-zlib php-mysqli php-curl php-xml php-devel\
#                 php-process php-soap php-opcache php-fpm ca-certificates \
#                 php-gmp php-pecl-xdebug php-pecl-zip mod_ssl hostname && \
#     wget https://getcomposer.org/installer -O /tmp/composer-installer.php && \
#     php /tmp/composer-installer.php --filename=composer --install-dir=/usr/local/bin

# ENV PHP_DEFAULT_INCLUDE_PATH=/usr/share/pear \
#     PHP_SYSCONF_PATH=/etc \
#     PHP_HTTPD_CONF_FILE=php.conf \
#     HTTPD_MAIN_CONF_PATH=/etc/httpd/conf \
#     HTTPD_MAIN_CONF_D_PATH=/etc/httpd/conf.d \
#     HTTPD_MODULES_CONF_D_PATH=/etc/httpd/conf.modules.d \
#     HTTPD_VAR_RUN=/var/run/httpd \
#     HTTPD_DATA_PATH=/var/www \
#     HTTPD_DATA_ORIG_PATH=/var/www \
#     HTTPD_VAR_PATH=/var

# .NET
ENV DOTNET_RPM_VERSION=8.0
RUN dnf install -y dotnet-hostfxr-${DOTNET_RPM_VERSION} dotnet-runtime-${DOTNET_RPM_VERSION} dotnet-sdk-${DOTNET_RPM_VERSION}

# rust
# ENV CARGO_HOME=/home/tooling/.cargo \
#     RUSTUP_HOME=/home/tooling/.rustup \
#     PATH=/home/tooling/.cargo/bin:${PATH}
# RUN curl --proto '=https' --tlsv1.2 -sSfo rustup https://sh.rustup.rs && \
#     chmod +x rustup && \
#     mv rustup /usr/bin/ && \
#     rustup -y --no-modify-path --profile minimal -c rust-src -c rust-analysis -c rls && \
#     chgrp -R 0 /home/tooling && chmod -R g=u /home/tooling

# camel-k
ENV KAMEL_VERSION 2.2.0
RUN curl -L https://github.com/apache/camel-k/releases/download/v${KAMEL_VERSION}/camel-k-client-${KAMEL_VERSION}-linux-amd64.tar.gz | tar -C /usr/local/bin -xz --no-same-owner \
    && chmod +x /usr/local/bin/kamel

# Config directories
RUN mkdir -p /home/tooling/.m2 && \
    mkdir -p /home/tooling/.gradle && \
    mkdir -p /home/tooling/.config/pip && \
    mkdir -p /home/tooling/.sbt/1.0 && \
    mkdir -p /home/tooling/.cargo && \
    mkdir -p /home/tooling/certs && \
    mkdir -p /home/tooling/.composer && \
    mkdir -p /home/tooling/.nuget && \
    chgrp -R 0 /home/tooling && chmod -R g=u /home/tooling

# Cloud

# oc client
ENV OC_VERSION=4.15
RUN curl -L https://mirror.openshift.com/pub/openshift-v4/arm64/clients/ocp/stable-${OC_VERSION}/openshift-client-linux.tar.gz | tar -C /usr/local/bin -xz --no-same-owner \
    && chmod +x /usr/local/bin/oc

# OS Pipelines CLI (tkn)
ENV TKN_VERSION=1.14.0
RUN curl -L https://mirror.openshift.com/pub/openshift-v4/clients/pipelines/${TKN_VERSION}/tkn-linux-amd64.tar.gz | tar -C /usr/local/bin -xz --no-same-owner \
    && chmod +x /usr/local/bin/tkn /usr/local/bin/opc /usr/local/bin/tkn-pac

RUN echo 'alias docker=podman' >> ${PROFILE_EXT}

# Configure container engine
COPY --chown=0:0 containers.conf /etc/containers/containers.conf

ENV K8S_VERSION=1.28
RUN curl -LO "https://dl.k8s.io/release/v${K8S_VERSION}.0/bin/linux/arm64/kubectl" && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/kubectl && \
    curl -sSL -o ~/.kubectl_aliases https://raw.githubusercontent.com/ahmetb/kubectl-alias/master/.kubectl_aliases && \
    echo '[ -f ~/.kubectl_aliases ] && source ~/.kubectl_aliases' >> ${PROFILE_EXT}

## shellcheck
RUN dnf install -y xz && \
    curl -sSL "https://github.com/koalaman/shellcheck/releases/download/v0.10.0/shellcheck-v0.10.0.linux.armv6hf.tar.xz" | tar -xJv && \
    mv shellcheck-v0.10.0/shellcheck /usr/local/bin/ && \
    chmod +x /usr/local/bin/shellcheck && \
    rm -rf shellcheck-v0.10.0

## krew
RUN set -euf -o pipefail && \
    TEMP_DIR="$(mktemp -d)" && \
    cd "${TEMP_DIR}" && \
    KREW_VERSION="0.4.4" && \
    KREW_ARCH="linux_arm64" && \
    KREW_TGZ="krew-${KREW_ARCH}.tar.gz" && \
    KREW_TGZ_URL="https://github.com/kubernetes-sigs/krew/releases/download/v${KREW_VERSION}/${KREW_TGZ}" && \
    curl -sSLO "${KREW_TGZ_URL}" && \
    tar -zxv --no-same-owner -f "${KREW_TGZ}" && \
    ./"krew-${KREW_ARCH}" install krew && \
    echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ${PROFILE_EXT} && \
    export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH" && \
    kubectl krew install ns && \
    kubectl krew install ctx && \
    cd - && \
    rm -rf "${TEMP_DIR}"

## helm
RUN curl -sSL "https://get.helm.sh/helm-v3.14.3-linux-arm64.tar.gz" | tar -xz && \
    mv linux-arm64/helm /usr/local/bin/helm && \
    chmod +x /usr/local/bin/helm && \
    rm -rf linux-arm64

## kustomize
RUN curl -sSL "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize/v5.3.0/kustomize_v5.3.0_linux_arm64.tar.gz" | tar -xz && \
    mv kustomize /usr/local/bin/kustomize && \
    chmod +x /usr/local/bin/kustomize

## tektoncd-cli
RUN curl -sSL -o tektoncd-cli.rpm "https://github.com/tektoncd/cli/releases/download/v0.39.0/tektoncd-cli-0.39.0_Linux-ARM64.rpm" && \
    dnf install -y tektoncd-cli.rpm && \
    rm tektoncd-cli.rpm

## knative-cli
RUN curl -sSL "https://github.com/knative/client/releases/download/knative-v1.13.0/kn-linux-arm64" -o kn && \
    chmod +x kn && \
    mv kn /usr/local/bin/kn

## terraform-cli
RUN curl -sSL "https://releases.hashicorp.com/terraform/1.7.5/terraform_1.7.5_linux_arm64.zip" -o terraform.zip && \
    unzip terraform.zip && \
    chmod +x terraform && \
    mv terraform /usr/local/bin/terraform && \
    rm terraform.zip

# Install Terraform-docs
RUN curl -sSLo ./terraform-docs.tar.gz https://terraform-docs.io/dl/v0.19.0/terraform-docs-v0.19.0-linux-arm64.tar.gz; \
    tar -xzf terraform-docs.tar.gz; \
    chmod +x terraform-docs; \
    mv terraform-docs /usr/local/bin/terraform-docs

# ## skaffold
# RUN curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && \
#     install skaffold /usr/local/bin/

# # e2fsprogs setup
# # Since e2fsprogs-static package has removed RHEL 8 distribution, it is not possible to install from the repository
# # https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/considerations_in_adopting_rhel_8/index#removed-packages_changes-to-packages
# RUN <<EOF
# set -euf -o pipefail
# TEMP_DIR="$(mktemp -d)"
# cd "${TEMP_DIR}"
# E2FSPROGS_VERSION="1.46.5"
# E2FSPROGS_TGZ="e2fsprogs-${E2FSPROGS_VERSION}.tar.gz"
# E2FSPROGS_TGZ_URL="https://mirrors.edge.kernel.org/pub/linux/kernel/people/tytso/e2fsprogs/v${E2FSPROGS_VERSION}/${E2FSPROGS_TGZ}"
# E2FSPROGS_CHEKSUMS_URL="https://mirrors.edge.kernel.org/pub/linux/kernel/people/tytso/e2fsprogs/v${E2FSPROGS_VERSION}/sha256sums.asc"
# curl -sSLO "${E2FSPROGS_TGZ_URL}"
# curl -sSLO "${E2FSPROGS_CHEKSUMS_URL}"
# sha256sum --ignore-missing -c "sha256sums.asc" 2>&1 | grep OK
# tar -zxv --no-same-owner -f "${E2FSPROGS_TGZ}"
# cd "e2fsprogs-${E2FSPROGS_VERSION}"
# mkdir build
# cd build
# ../configure --prefix=/usr --with-root-prefix="" --enable-elf-shlibs --disable-evms
# make
# make install
# make install-libs
# cd -
# rm -rf "${TEMP_DIR}"
# EOF

# Bash completions
# RUN dnf -y install bash-completion && \
#     oc completion bash > /usr/share/bash-completion/completions/oc && \
#     chmod +x /usr/local/bin/tkn && \
#     tkn completion bash > /usr/share/bash-completion/completions/tkn && \
#     kubectl completion bash > /usr/share/bash-completion/completions/kubectl && \
#     cat ${NVM_DIR}/bash_completion > /usr/share/bash-completion/completions/nvm && \
#     dnf clean all && \
#     rm -rf /var/cache/yum

## Add sdkman's init script launcher to the end of ${PROFILE_EXT} since we are not adding it on sdkman install
## NOTE: all modifications to ${PROFILE_EXT} must happen BEFORE this step in order for sdkman to function correctly
# RUN echo 'export SDKMAN_DIR="/home/tooling/.sdkman"' >> ${PROFILE_EXT}
# RUN echo '[[ -s "$SDKMAN_DIR/bin/sdkman-init.sh" ]] && source "$SDKMAN_DIR/bin/sdkman-init.sh"' >> ${PROFILE_EXT}

USER 0

# Ensure EPEL repository is properly configured
RUN dnf install -y epel-release && \
    dnf update -y && \
    dnf install -y yum-utils zsh iputils && \
    dnf install bind-utils -y && \
    dnf -y install bash-completion && \
    pip install ansible && \
    rm -rf /var/cache/yum

# Install Oh-My-Zsh
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.2.1/zsh-in-docker.sh)" -- \
    -t jonathan \
    -p ansible \
    -p terraform \
    -p kubectl \
    -p helm \
    -p oc \
    -p podman \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions

# Install Ansible, Zsh and iputils
RUN yum install -y yum-utils zsh iputils && \
    dnf install bind-utils -y && \
    dnf -y install bash-completion && \
    pip install ansible && \
    rm -rf /var/cache/yum

# Install Terraform-docs
RUN curl -sSLo ./terraform-docs.tar.gz https://terraform-docs.io/dl/v0.19.0/terraform-docs-v0.19.0-linux-arm64.tar.gz; \
    tar -xzf terraform-docs.tar.gz; \
    chmod +x terraform-docs; \
    mv terraform-docs /usr/local/bin/terraform-docs

RUN yum install -y yum-utils zsh && \
    pip install ansible

# Add Microsoft repository for PowerShell
RUN dnf install -y openssl libunwind && \
    mkdir -p /opt/microsoft/powershell/7 && \
    wget -O -q /tmp/powershell.tar.gz https://github.com/PowerShell/PowerShell/releases/download/v7.5.0-rc.1/powershell-7.5.0-rc.1-linux-arm64.tar.gz && \
    tar zxf /tmp/powershell.tar.gz -C /opt/microsoft/powershell/7 && \
    chmod +x /opt/microsoft/powershell/7/pwsh && \
    ln -s /opt/microsoft/powershell/7/pwsh /usr/bin/pwsh

# Install locales package and generate the locale
RUN dnf install -y glibc-langpack-en glibc-all-langpacks && \
    dnf reinstall -y glibc-common && \
    dnf clean all && \
    echo "LANG=en_US.UTF-8" > /etc/locale.conf && \
    localedef -v -c -i en_US -f UTF-8 en_US.UTF-8 || true

# Set environment variables for locale
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8 

# Disable oh-my-zsh compfix verification
ENV ZSH_DISABLE_COMPFIX=true

# Install lastest AWS CLI
RUN yum remove awscli -y; \
    curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"; \
    unzip -q awscliv2.zip; \
    sudo ./aws/install; \
    rm -rf awscliv2.zip aws

# Create symbolic links from /home/tooling/ -> /home/user/
RUN stow . -t /home/user/ -d /home/tooling/ --no-folding

# Set permissions on /etc/passwd, /etc/group, /etc/pki and /home to allow arbitrary users to write
RUN chgrp -R 0 /home && chmod -R g=u /etc/passwd /etc/group /home /etc/pki

# cleanup dnf cache
RUN dnf -y clean all --enablerepo='*'

COPY --chown=0:0 entrypoint.sh /

USER 10001

ENV SHELL=/bin/zsh

ENV HOME=/home/user


