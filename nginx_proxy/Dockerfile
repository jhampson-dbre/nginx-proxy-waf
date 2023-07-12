ARG BUILD_FROM
ARG BUILD_ARCH
ARG BUILD_MODSECURITY_VERSION
ARG NGINX_VERSION

FROM jhampdbre/${BUILD_ARCH}-nginx-${NGINX_VERSION}-modsecurity:${BUILD_MODSECURITY_VERSION} as build_modsecurity

FROM $BUILD_FROM

ARG NGINX_VERSION
ARG PKG_RELEASE

# Setup base
RUN set -x \
    && KEY_SHA512="e09fa32f0a0eab2b879ccbbc4d0e4fb9751486eedda75e35fac65802cc9faa266425edf83e261137a2f4d16281ce2c1a5f4502930fe75154723da014214f0655" \
    && apk upgrade --no-cache \
    && apk add --no-cache \
    geoip \
    libmaxminddb \
    libstdc++ \
    libxml2 \
    lmdb \
    yajl \
    openssl \
    pcre \
    pcre2 \
    && wget -O /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub \
    && if echo "$KEY_SHA512 */tmp/nginx_signing.rsa.pub" | sha512sum -c -; then \
    echo "key verification succeeded!"; \
    mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/; \
    else \
    echo "key verification failed!"; \
    exit 1; \
    fi \
    && ALPINE_VERSION=$(egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release) \
    && apk add -X "https://nginx.org/packages/alpine/v${ALPINE_VERSION}/main" --no-cache nginx=${NGINX_VERSION}-r${PKG_RELEASE} \
    && mkdir /etc/nginx/modsec

# Copy owasp-modsecurity-crs and modsecurity from the build image
COPY --from=build_modsecurity /usr/local/modsecurity /usr/local/modsecurity
COPY --from=build_modsecurity /usr/local/owasp-modsecurity-crs /usr/local/owasp-modsecurity-crs
COPY --from=build_modsecurity /usr/lib/nginx/modules/ /usr/lib/nginx/modules/

# Copy data
COPY rootfs /

CMD [ "/run.sh" ]
