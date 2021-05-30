FROM debian:buster-slim

# Upgrade OS and install needed packages
RUN apt-get update -y && apt-get upgrade -y
RUN apt-get install -y build-essential \
    curl bash sudo vim git wget apt-utils base zsh \
    less lsb-release ruby locales ruby-dev \
    zlib1g-dev software-properties-common

# Make sure the locale is set up correctly
RUN dpkg-reconfigure locales && \
    locale-gen C.UTF-8 && \
    /usr/sbin/update-locale LANG=C.UTF-8

# Install needed default locale for Makefly
RUN echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && \
    locale-gen

# Set default locale for the environment
ENV LC_ALL C.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8


# Install Docker CLI
ENV DOCKER_VERSION 20.10.6
RUN curl "https://download.docker.com/linux/static/stable/$(arch)/docker-${DOCKER_VERSION}.tgz" -o docker.tgz && \
    tar zxvf docker.tgz && \
    mv ./docker/docker /usr/bin/docker && \
    rm -Rf docker

# Create VSCode development user
RUN groupadd -g 1000 vscode && useradd -m -u 1000 -G sudo -g vscode -s /bin/zsh vscode
RUN echo "# zsh config" >> /home/vscode/.zshrc
RUN echo "vscode:vscode" | chpasswd
RUN echo "vscode ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Install Jekyll dependencies
RUN gem install jekyll bundler jekyll-gist jekyll-sitemap jekyll-seo-tag

# Switch to container non-root user
USER vscode

# Set up zsh
RUN wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh
RUN cp /home/vscode/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
RUN echo "alias ls=\"ls -alh\"" >> /home/vscode/.zshrc