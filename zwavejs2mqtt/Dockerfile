ARG BUILD_FROM=ghcr.io/hassio-addons/base:12.2.0
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

COPY node-zwave-js_zl7432.patch /tmp/

# Setup base
# hadolint ignore=DL3003,SC2046
RUN \
    apk add --no-cache --virtual .build-dependencies \
        build-base=0.5-r3 \
        linux-headers=5.16.7-r1 \
        npm=8.10.0-r0 \
        python3-dev=3.10.4-r0 \
    \
    && apk add --no-cache \
        eudev=3.2.11-r0 \
        libusb=1.0.26-r0 \
        nginx=1.22.0-r1 \
        nodejs=16.15.0-r1 \
    \
    && npm install --global yarn@2.4.3 \
    \
    && curl -J -L -o /tmp/zwavejs2mqtt.tar.gz \
        $(curl -s https://api.github.com/repos/zwave-js/zwavejs2mqtt/releases/latest | jq -r '.tarball_url') \
    && tar zxvf \
        /tmp/zwavejs2mqtt.tar.gz \
        --strip 1 -C /opt \
    \
    && cd /opt \
    && yarn install --immutable \
    && yarn run build:server \
    && yarn run build:ui \
    && npm_config_build_from_source=true \
        yarn rebuild @serialport/bindings-cpp \
    && yarn remove \
        $(jq -r '.devDependencies | keys | join(" ")' package.json) \
    && export ZWJS="https://github.com/zwave-js/node-zwave-js/archive/refs/tags/v$(node -p "require('/opt/node_modules/zwave-js/package.json').version").tar.gz" \
    && mkdir /tmp/node-zwave-js \
    && curl -J -L -o /tmp/node-zwave-js.tar.gz ${ZWJS} \
    && tar zxvf \
        /tmp/node-zwave-js.tar.gz \
        --strip 1 -C /tmp/node-zwave-js \
    \
    && cd /tmp/node-zwave-js \
    && patch -p0  < /tmp/node-zwave-js_zl7432.patch \
    \
    && yarn \
    && yarn run build \
    && ls -l /tmp/node-zwave-js \
    && ls -l /tmp/node-zwave-js/node_modules \
    && cp -v /tmp/node-zwave-js/packages/config/config/devices/0x0109/zl7432.json /opt/node_modules/@zwave-js/config/config/devices/0x0109/zl7432.json \
    && cp -v /tmp/node-zwave-js/packages/config/build/devices/CompatConfig* /opt/node_modules/@zwave-js/config/build/devices/ \
    && cp -v /tmp/node-zwave-js/packages/zwave-js/build/lib/node/Node.* /opt/node_modules/zwave-js/build/lib/node/ \
    && yarn cache clean \
    \
    && npm uninstall -g yarn \
    && apk del --no-cache --purge .build-dependencies \
    && rm -f -r \
        /etc/nginx \
        /opt/build \
        /opt/index.html \
        /opt/package.sh \
        /opt/src \
        /opt/static \
        /opt/stylesheets \
        /opt/.[!.]* \
        /opt/*.md \
        /opt/nodemon.json \
        /opt/kustomization.yaml \
        /opt/kubernetes \
        /opt/docker \
        /opt/docs \
        /root/.cache \
        /root/.config \
        /root/.npmrc \
        /root/.node-gyp \
        /root/.npm \
        /tmp/.[!.]* \
        /tmp/* \
        /usr/lib/node_modules \
        /usr/local/share/.cache

# Copy root filesystem
COPY rootfs /

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
