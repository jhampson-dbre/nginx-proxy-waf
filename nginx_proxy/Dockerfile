ARG BUILD_FROM
ARG BUILD_ARCH
ARG BUILD_MODSECURITY_VERSION
ARG NGINX_VERSION
ARG PKG_RELEASE

FROM jhampdbre/${BUILD_ARCH}-nginx-${NGINX_VERSION}-modsecurity:${BUILD_MODSECURITY_VERSION} as build_modsecurity

FROM $BUILD_FROM

# Setup base
RUN set -x \
    && KEY_SHA512="e7fa8303923d9b95db37a77ad46c68fd4755ff935d0a534d26eba83de193c76166c68bfe7f65471bf8881004ef4aa6df3e34689c305662750c0172fca5d8552a *stdin" \
    && apk add --no-cache \
    geoip \
    libmaxminddb \
    libstdc++ \
    libxml2 \
    lmdb \
    yajl \
    openssl \
    && wget -O /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub \
    && if [ "$(openssl rsa -pubin -in /tmp/nginx_signing.rsa.pub -text -noout | openssl sha512 -r)" = "$KEY_SHA512" ]; then \
    echo "key verification succeeded!"; \
    mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/; \
    else \
    echo "key verification failed!"; \
    exit 1; \
    fi \
    && apk add -X "https://nginx.org/packages/alpine/v$(egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release)/main" --no-cache ${NGINX_VERSION}-r${PKG_RELEASE} \
    && mkdir /etc/nginx/modsec

# Copy owasp-modsecurity-crs and modsecurity from the build image
COPY --from=build_modsecurity /usr/local/modsecurity /usr/local/modsecurity
COPY --from=build_modsecurity /usr/local/owasp-modsecurity-crs /usr/local/owasp-modsecurity-crs
COPY --from=build_modsecurity /usr/lib/nginx/modules/ /usr/lib/nginx/modules/

# Copy data
COPY rootfs /

CMD [ "/run.sh" ]
