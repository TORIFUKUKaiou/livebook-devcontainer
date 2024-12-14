# Update the VARIANT arg in docker-compose.yml to pick a Ubuntu version
ARG VARIANT="jammy-20240911.1"
FROM ubuntu:${VARIANT}

# This Dockerfile adds a non-root user with sudo access. Update the “remoteUser” property in
# devcontainer.json to use it. More info: https://aka.ms/vscode-remote/containers/non-root-user.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Options for common package install script
ARG INSTALL_ZSH="false"
ARG UPGRADE_PACKAGES="true"
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/script-library/common-debian.sh"
ARG COMMON_SCRIPT_SHA="dev-mode"

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
RUN apt-get update \
  && export DEBIAN_FRONTEND=noninteractive \
  && apt-get -y install --no-install-recommends curl ca-certificates 2>&1 \
  && curl -sSL ${COMMON_SCRIPT_SOURCE} -o /tmp/common-setup.sh \
  && ([ "${COMMON_SCRIPT_SHA}" = "dev-mode" ] || (echo "${COMMON_SCRIPT_SHA} */tmp/common-setup.sh" | sha256sum -c -)) \
  && /bin/bash /tmp/common-setup.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
  # Install dependencies
  && apt-get install -y build-essential inotify-tools git \
  #
  # Clean up
  && apt-get autoremove -y \
  && apt-get clean -y \
  && rm -rf /var/lib/apt/lists/* /tmp/common-setup.sh

RUN su ${USERNAME} -c "mkdir -p /home/vscode/.elixir-install"
RUN su ${USERNAME} -c "curl -fsSo '/home/vscode/.elixir-install/install.sh' 'https://elixir-lang.org/install.sh'"

ARG ELIXIR_VERSION='1.17.3'
ARG ELIXIR_OTP_RELEASE='27'
ARG OTP_VERSION='27.2'

RUN su ${USERNAME} -c "sh /home/vscode/.elixir-install/install.sh elixir@${ELIXIR_VERSION} otp@${OTP_VERSION}"
RUN su ${USERNAME} -c "echo 'export PATH=/home/vscode/.elixir-install/installs/otp/${OTP_VERSION}/bin:/home/vscode/.elixir-install/installs/elixir/${ELIXIR_VERSION}-otp-${ELIXIR_OTP_RELEASE}/bin:$PATH' >> /home/vscode/.profile"

RUN su ${USERNAME} -c "source /home/vscode/.profile \
  && mix local.hex --force \
  && mix local.rebar --force"

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update \
#     && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment this line to install additional package.
# RUN  mix ...

ARG LIVEBOOK_VERSION="v0.14.5"

WORKDIR /home/vscode

RUN su ${USERNAME} -c "git clone https://github.com/livebook-dev/livebook.git \
  && cd livebook \
  && git checkout -b tags/${LIVEBOOK_VERSION} ${LIVEBOOK_VERSION} \
  && source /home/vscode/.profile \
  && mix deps.get --only prod"
