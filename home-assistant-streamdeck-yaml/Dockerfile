ARG BUILD_FROM=ghcr.io/hassio-addons/base:17.2.5
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Build arguments
ARG BUILD_ARCH=amd64
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION
ARG VERSION=2026.5.2.post3
# Fork van basnijholt/home-assistant-streamdeck-yaml met o.a. een foto-screensaver.
# Een vaste commit, zodat elke installatie precies dezelfde code krijgt.
ARG REPO=tjah2/home-assistant-streamdeck-yaml
ARG COMMIT=405f6a5e124c47b1ce91cf07e3d48defae49720e

# Material Design Icons: alle iconen zitten in het image (assets/mdi.zip), zodat
# de add-on ze niet bij elke start opnieuw van GitHub hoeft te downloaden (Home
# Assistant maakt de container bij elke start opnieuw aan) en ook zonder
# internet werkt. De library leest ze uit de zip (zie MDI_ARCHIVE).
ARG MDI_VERSION=7.4.47

# Python-pakketten komen kant-en-klaar uit Alpine (geen gcc, niets compileren)
# en delen de systeembibliotheken (cairo, jpeg, png, freetype, libxml2).
# Via pip alleen wat Alpine niet (in de juiste versie) heeft: streamdeck,
# pydantic 1.x en websockets (Alpine 3.21 heeft 14.1, de library wil nieuwer).
# hadolint ignore=DL3018,DL3003
RUN apk --no-cache add \
    python3 \
    libusb \
    hidapi \
    py3-cairosvg \
    py3-dotenv \
    py3-jinja2 \
    py3-lxml \
    py3-pillow \
    py3-requests \
    py3-rich \
    py3-yaml \
    && apk add --no-cache --virtual .build-deps \
    py3-pip \
    # Download and install the application
    && echo "Downloading ${REPO} at ${COMMIT} (version ${VERSION})..." \
    && wget -nv https://github.com/${REPO}/archive/${COMMIT}.zip -O /tmp/app.zip \
    && unzip -q /tmp/app.zip -d /tmp \
    && mv /tmp/home-assistant-streamdeck-yaml-${COMMIT} /app \
    && rm -r /tmp/app.zip /app/tests \
    && echo "Downloading Material Design Icons ${MDI_VERSION}..." \
    && wget -nv https://registry.npmjs.org/@mdi/svg/-/svg-${MDI_VERSION}.tgz -O /tmp/mdi.tgz \
    # One zip instead of 7447 small files (3 MB instead of 30 MB on disk)
    && python3 -c "import tarfile, zipfile; \
tar = tarfile.open('/tmp/mdi.tgz'); \
out = zipfile.ZipFile('/app/assets/mdi.zip', 'w', zipfile.ZIP_DEFLATED, compresslevel=9); \
[out.writestr(m.name.split('/')[-1], tar.extractfile(m).read()) for m in tar \
 if m.isfile() and m.name.startswith('package/svg/') and m.name.endswith('.svg')]; \
out.close()" \
    && rm /tmp/mdi.tgz \
    # Install the application (the Alpine packages above already satisfy most dependencies)
    && cd /app \
    && SETUPTOOLS_SCM_PRETEND_VERSION=${VERSION} pip install --no-cache-dir --break-system-packages -e "." \
    && python3 -m compileall -q /app \
    && python3 -c "import home_assistant_streamdeck_yaml, cairosvg, StreamDeck, websockets; websockets.ClientConnection" \
    # pip is only needed to install; remove it and the pip/setuptools leftovers
    && apk del .build-deps \
    && find /usr/lib/python3.12/site-packages -maxdepth 1 \( -name "pip" -o -name "pip-*" \) -exec rm -rf {} +

# Copy rootfs
COPY rootfs /

# Labels
LABEL \
    io.hass.name="home-assistant-streamdeck-yaml" \
    io.hass.description="home-assistant-streamdeck-yaml add-on" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Bas Nijholt <bas@nijho.lt>" \
    org.label-schema.description="home-assistant-streamdeck-yaml add-on" \
    org.label-schema.build-date=${BUILD_DATE} \
    org.label-schema.name="home-assistant-streamdeck-yaml" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.url="https://addons.community" \
    org.label-schema.usage="https://github.com/basnijholt/home-assistant-streamdeck-yaml-addon/tree/main/README.md" \
    org.label-schema.vcs-ref=${BUILD_REF} \
    org.label-schema.vcs-url="https://github.com/basnijholt/home-assistant-streamdeck-yaml-addon" \
    org.label-schema.vendor="Bas Nijholt"
