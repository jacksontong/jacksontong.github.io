FROM mcr.microsoft.com/devcontainers/ruby:3.3-bookworm

RUN gem install jekyll bundler

USER vscode

RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.2.0/zsh-in-docker.sh)" -- \
    -p git \
    -p https://github.com/zsh-users/zsh-autosuggestions
