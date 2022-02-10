ARG BUILD_FROM

FROM ${BUILD_FROM}

ARG NGINX_VERSION
ARG MODSEC_BRANCH
ARG OWASP_BRANCH

WORKDIR /opt

# Install dependencies; includes dependencies required for compile-time options:
# curl, libxml, pcre, and lmdb and Modsec
# https://build.alpinelinux.org/buildlogs/build-3-11-x86_64/main/nginx/nginx-1.16.1-r6.log
RUN echo "Installing Dependencies" && \
    apk add --no-cache --virtual general-dependencies \
    alpine-sdk \
    autoconf \
    automake \
    byacc \
    curl-dev \
    findutils \
    flex \
    g++ \
    gcc \
    gd-dev \
    geoip-dev \
    git \
    libc-dev \
    libedit-dev \
    libstdc++ \
    libtool \
    libxml2-dev \
    libxslt-dev \
    linux-headers \
    lmdb-dev \
    make \
    libmaxminddb-dev \
    openssl-dev \
    pcre-dev \
    perl-dev \
    yajl-dev \
    zlib-dev

# Clone and compile modsecurity. Binary will be located in /usr/local/modsecurity
RUN echo "Installing ModSec Library" && \
    git clone -b ${MODSEC_BRANCH} --depth 1 https://github.com/SpiderLabs/ModSecurity && \
    git -C /opt/ModSecurity submodule update --init --recursive && \
    (cd "/opt/ModSecurity" && \
    ./build.sh && \
    ./configure && \
    make -j4 && \
    make install \
    ) && \
    rm -fr /opt/ModSecurity \
    /usr/local/modsecurity/lib/libmodsecurity.a \
    /usr/local/modsecurity/lib/libmodsecurity.la

# Clone Modsec Nginx Connector, ModSec OWASP Rules, and download/extract nginx and GeoIP databases
RUN echo 'Cloning Modsec Nginx Connector and ModSec OWASP Rules' && \
    git clone -b master --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git && \
    git clone -b ${OWASP_BRANCH} --depth 1 https://github.com/coreruleset/coreruleset.git /usr/local/owasp-modsecurity-crs && \
    wget -O - https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz | tar -xz

# Install ModSecurity Nginx modules
RUN echo 'Installing Nginx Modules' && \
    (cd "/opt/nginx-${NGINX_VERSION}" && \
    ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx \
    && make modules \
    ) && \
    mkdir -p /usr/lib/nginx/modules && \
    cp /opt/nginx-${NGINX_VERSION}/objs/ngx_http_modsecurity_module.so \
    /usr/lib/nginx/modules/ && \
    rm -fr /opt/* && \
    apk del general-dependencies