ARG BASE_IMAGE=ubuntu:22.04
FROM ${BASE_IMAGE}

ARG PROJECT_NAME=ascender
ARG USER_NAME=challenger
ARG GROUP_NAME=challengers
ARG UID=1000
ARG GID=1000
ARG PYTHON_VERSION=3.12
ARG APPLICATION_DIRECTORY=/home/${USER_NAME}/${PROJECT_NAME}
ARG UV_TOOL_BIN_DIRECTORY=/home/${USER_NAME}/.local/bin/
ARG INSTALL_UV_TOOLS="true"
ARG RUN_UV_SYNC_AT_BUILD_TIME="false"

# To use uv tools, UV_TOOL_BIN_DIRECTORY must be added to PATH.
ENV DEBIAN_FRONTEND="noninteractive" \
    LC_ALL="C.UTF-8" \
    LANG="C.UTF-8" \
    PYTHONPATH=${APPLICATION_DIRECTORY} \
	UV_TOOL_BIN_DIR=${UV_TOOL_BIN_DIRECTORY} \
	PATH=${UV_TOOL_BIN_DIRECTORY}:${PATH}

RUN apt update -y && apt install --no-install-recommends -y software-properties-common gpg-agent \
    # Add deadsnakes PPA to install older Python versions
    && add-apt-repository ppa:deadsnakes/ppa \
    && apt-get update && apt-get install -y \
    # Install specified Python
    python${PYTHON_VERSION} python${PYTHON_VERSION}-dev python3-pip python3-dev python-is-python3 \
    build-essential openssl libssl-dev libffi-dev ca-certificates git curl ssh openssh-client \
    && update-ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Swtich default python3 version
RUN update-alternatives --install /usr/bin/python python /usr/bin/python${PYTHON_VERSION} 1 \
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PYTHON_VERSION} 1

# Install pip for specified Python version
RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python${PYTHON_VERSION}

# Install Poethepoet directly using pip to type "poe" command without "uv run".
RUN python3 -m pip install --upgrade pip setuptools requests wheel six urllib3 \
    && python3 -m pip install uv poethepoet

# Add user. Without this, following process is executed as admin.
RUN groupadd -g ${GID} ${GROUP_NAME} \
    && useradd -ms /bin/sh -u ${UID} -g ${GID} ${USER_NAME}

USER ${USER_NAME}
WORKDIR ${APPLICATION_DIRECTORY}

COPY --chown=${UID}:${GID} pyproject.toml uv.lock  ./
RUN test ${INSTALL_UV_TOOLS} = "true" \
&& uv tool install ruff \
&& uv tool install mypy \
&& uv tool install pytest --with pytest-cov \
&& uv tool install mdformat --with mdformat-frontmatter --with mdformat-gfm --with mdformat-footnote \
|| echo "skip to install uv tools."

# If ${RUN_UV_SYNC_AT_BUILD_TIME} = "true", install Python package by uv and move .venv under ${HOME}.
# This process is for CI (GitHub Actions). To prevent overwrite by volume of docker compose, .venv is moved under ${HOME}.
RUN test ${RUN_UV_SYNC_AT_BUILD_TIME} = "true" && uv sync || echo "skip to run uv sync."
RUN test ${RUN_UV_SYNC_AT_BUILD_TIME} = "true" && mv ${APPLICATION_DIRECTORY}/.venv ${HOME}/.venv || echo "skip to move .venv."
