ARG BUILD_FROM
FROM $BUILD_FROM

ARG EUFY_SECURITY_WS_VERSION
ARG EUFY_SECURITY_CLIENT_REPOSITORY
ARG EUFY_SECURITY_CLIENT_REF

WORKDIR /usr/src/app
COPY package.json /usr/src/app/
RUN \
    set -x \
    && apk add --no-cache git \
    # The Alpine 3.23 base image (see build.yaml) ships Node.js 24 in its main
    # repository, so nodejs/npm install directly with no edge-repo workarounds.
    && apk add --no-cache \
        jq \
        nodejs \
        npm \
    && node --version \
    # Fail the build if the base ever regresses below Node 24.
    && case "$(node --version)" in v24.*) ;; *) echo "Expected Node 24, got $(node --version)" >&2; exit 1 ;; esac \
    && git clone --filter=blob:none --no-checkout "$EUFY_SECURITY_CLIENT_REPOSITORY" /usr/src/client \
    && cd /usr/src/client \
    && git fetch --depth 1 origin "$EUFY_SECURITY_CLIENT_REF" \
    && git checkout --detach FETCH_HEAD \
    && npm ci \
    && npm run build \
    && CLIENT_ARCHIVE="$(npm pack --pack-destination /tmp --silent)" \
    && mv "/tmp/${CLIENT_ARCHIVE}" /usr/src/app/eufy-security-client.tgz \
    && cd /usr/src/app \
    && npm install --omit=dev --force \
    && test "$(node -p 'require("/usr/src/app/node_modules/eufy-security-client/package.json").version')" = "4.1.1" \
    && test "$(node -p 'require("/usr/src/app/node_modules/eufy-security-ws/package.json").version')" = "$EUFY_SECURITY_WS_VERSION" \
    && rm -rf /usr/src/client \
    && npm cache clean --force

COPY run.sh /
RUN chmod a+x /run.sh
WORKDIR /data

CMD [ "/run.sh" ]
