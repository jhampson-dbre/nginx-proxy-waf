ARG BUILD_FROM
ARG BUILD_ARCH
ARG BUILD_MODSECURITY_VERSION

FROM jhampdbre/${BUILD_ARCH}-nginx-116-modsecurity:${BUILD_MODSECURITY_VERSION} as build_modsecurity

FROM $BUILD_FROM

# Setup base
RUN apk add --no-cache \
    geoip \
    libmaxminddb \
    libstdc++ \
    libxml2 \
    lmdb \
    yajl \
    nginx \
    openssl \
    && mkdir /etc/nginx/modsec

# Copy owasp-modsecurity-crs and modsecurity from the build image
COPY --from=build_modsecurity /usr/local/modsecurity /usr/local/modsecurity
COPY --from=build_modsecurity /usr/local/owasp-modsecurity-crs /usr/local/owasp-modsecurity-crs
COPY --from=build_modsecurity /usr/lib/nginx/modules/ /usr/lib/nginx/modules/

# Copy data
COPY rootfs /

CMD [ "/run.sh" ]
