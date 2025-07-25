FROM ubuntu:24.04
ARG RUNNER_VERSION="2.324.0"
ARG DEBIAN_FRONTEND=noninteractive

# Update and upgrade the system
RUN apt update -y && apt upgrade -y

# Add a user named docker
RUN useradd -m docker

# Install necessary packages including sudo
RUN apt install -y --no-install-recommends \
    curl build-essential libssl-dev libffi-dev python3 python3-venv python3-dev python3-pip jq \
    sudo ssh libicu-dev zstd

# Give docker user sudo privileges without password
RUN echo "docker ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Set up the actions runner using a fork that allows local caching
RUN cd /home/docker && mkdir actions-runner && cd actions-runner \
    && curl -o actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz -L https://github.com/falcondev-oss/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz \
    && tar xzf actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz

# Install dependencies and copy start script
RUN /home/docker/actions-runner/bin/installdependencies.sh
COPY start.sh /start.sh
RUN chmod +x /start.sh

# Change ownership to docker user
RUN chown -R docker:docker /home/docker

# Add properly owned cache folder to docker directory for cache mount
RUN mkdir -p /home/docker/cache && chown -R docker:docker /home/docker/cache


# Switch to docker user once and stay there
USER docker
WORKDIR /home/docker

# Install rustup as the docker user
RUN curl https://sh.rustup.rs -sSf | \
    sh -s -- --default-toolchain stable -y

# Add cargo bin to PATH
ENV PATH="/home/docker/.cargo/bin:${PATH}"

# Define the entrypoint
ENTRYPOINT ["/start.sh"]