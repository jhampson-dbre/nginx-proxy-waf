ARG NGINX_VER=1.16.1
ARG BUILD_FROM

FROM $BUILD_FROM as build_modsecurity

ARG MODSEC_BRANCH=v3.0.4
ARG OWASP_BRANCH=v3.3/master

WORKDIR /opt

# Install dependencies; includes dependencies required for compile-time options:
# curl, libxml, pcre, and lmdb and Modsec
RUN echo "Installing Dependencies" && \
    apk add --no-cache --virtual general-dependencies \
    autoconf \
    automake \
    byacc \
    curl-dev \
    flex \
    g++ \
    gcc \
    git \
    libc-dev \
    libstdc++ \
    libtool \
    libxml2-dev \
    linux-headers \
    lmdb-dev \
    make \
    openssl-dev \
    pcre-dev \
    yajl-dev \
    zlib-dev

# Clone and compile modsecurity. Binary will be located in /usr/local/modsecurity
RUN echo "Installing ModSec Library" && \
    git clone -b ${MODSEC_BRANCH} --depth 1 https://github.com/SpiderLabs/ModSecurity && \
    git -C /opt/ModSecurity submodule update --init --recursive && \
    (cd "/opt/ModSecurity" && \
        ./build.sh && \
        ./configure && \
        make && \
        make install \
    ) && \
    rm -fr /opt/ModSecurity \
        /usr/local/modsecurity/lib/libmodsecurity.a \
        /usr/local/modsecurity/lib/libmodsecurity.la

# Clone Modsec Nginx Connector, ModSec OWASP Rules, and download/extract nginx and GeoIP databases
RUN echo 'Cloning Modsec Nginx Connector and ModSec OWASP Rules' && \
    git clone -b master --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git && \
    git clone -b ${OWASP_BRANCH} --depth 1 https://github.com/coreruleset/coreruleset.git /usr/local/owasp-modsecurity-crs && \
    wget -O - https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz | tar -xz

# Install ModSecurity Nginx modules
RUN echo 'Installing Nginx Modules' && \
    (cd "/opt/nginx-$NGINX_VERSION" && \
        ./configure --with-compat \
            --add-dynamic-module=../ModSecurity-nginx && \
        make modules \
    ) && \
    cp /opt/nginx-$NGINX_VERSION/objs/ngx_http_modsecurity_module.so \
        /usr/lib/nginx/modules/ && \
    rm -fr /opt/* && \
    apk del general-dependencies


FROM $BUILD_FROM

# Setup base
RUN apk add --no-cache \
    curl-dev \
    libstdc++ \
    libxml2-dev \
    lmdb-dev \
    tzdata \
    yajl \
    nginx \
    openssl \
    && mkdir /etc/nginx/modsec

# Copy owasp-modsecurity-crs and modsecurity from the build image
COPY --from=build_modsecurity /usr/local/modsecurity /usr/local/modsecurity
COPY --from=build_modsecurity /usr/local/owasp-modsecurity-crs /usr/local/owasp-modsecurity-crs
COPY --from=build_modsecurity /usr/lib/nginx/modules/ /usr/lib/nginx/modules/

# Copy data
COPY data/run.sh /
COPY data/nginx.conf /etc/
COPY data/modsec/ /etc/nginx/modsec/
COPY data/owasp/ /usr/local/owasp-modsecurity-crs/

CMD [ "/run.sh" ]
