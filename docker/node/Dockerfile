FROM node:latest

LABEL maintainer="Chris Fidao"

# we can also set it in 'docker-compose.yml' like this: 'working_dir:/opt', under the 'node' service declaration
#WORKDIR /opt

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb http://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y git yarn \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
