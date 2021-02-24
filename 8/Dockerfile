ARG NODE_VERSION=erbium

FROM node:${NODE_VERSION}-alpine as builder

RUN apk add --no-cache make pkgconfig gcc g++ python libx11-dev libxkbfile-dev

ARG version=latest
WORKDIR /home/theia
ADD ../$version.package.json ./package.json
ARG GITHUB_TOKEN

RUN yarn --pure-lockfile && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build && \
    yarn theia download:plugins && \
    yarn --production && \
    yarn autoclean --init && \
    echo *.ts >> .yarnclean && \
    echo *.ts.map >> .yarnclean && \
    echo *.spec.* >> .yarnclean && \
    yarn autoclean --force && \
    yarn cache clean && \
    rm -rf .cache

FROM node:${NODE_VERSION}-alpine

WORKDIR /home/theia
ENV HOME=/home/theia \
    JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk \
    PATH=$PATH:$JAVA_HOME/jre/bin:$JAVA_HOME/bin \
    LANG=C.UTF-8

# See : https://github.com/theia-ide/theia-apps/issues/34
RUN set -x; \
    echo -e "http://mirrors.aliyun.com/alpine/v3.13/main\nhttp://mirrors.aliyun.com/alpine/v3.13/community" > /etc/apk/repositories; \
    apk add --no-cache \
    bash \
    bash-completion \
    git \
    tig \
    git-bash-completion \
    git-lfs \
    openssl \
    curl \
    unzip \
    openjdk8 \
    maven \
    gradle \
    tree \
    ; \
    apk add --no-cache --virtual .build-deps tzdata; \
    echo "Asia/Shanghai" > /etc/timezone; \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    apk del --no-network .build-deps; \
    \
    export JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk; \
    export PATH=$PATH:$JAVA_HOME/jre/bin:$JAVA_HOME/bin; \
    export LANG=C.UTF-8; \
    \
    addgroup theia; \
    adduser -G theia -s /bin/bash -D theia; \
    \
    chmod g+rw /home; \
    mkdir -p /home/project; \
    chown -R theia:theia /home/theia; \
    chown -R theia:theia /home/project; \
    \
    echo "source /etc/profile" >> /home/theia/.bashrc;

COPY --from=builder --chown=theia:theia /home/theia /home/theia

ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins \
    USE_LOCAL_GIT=true

EXPOSE 3000

USER theia
ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]