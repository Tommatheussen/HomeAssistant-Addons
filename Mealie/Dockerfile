# ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base-debian:stretch
ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base-python:3.9-alpine3.13
###############################################################################
# Get prebuild containers from Mealie
###############################################################################
FROM "hkotel/mealie:dev" as mealie

FROM $BUILD_FROM

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Mealie base image
ENV MEALIE_HOME="/app"

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=off \
    PIP_DISABLE_PIP_VERSION_CHECK=on \
    PIP_DEFAULT_TIMEOUT=100 \
    POETRY_HOME="/opt/poetry" \
    POETRY_VIRTUALENVS_IN_PROJECT=true \
    POETRY_NO_INTERACTION=1 \
    PYSETUP_PATH="/opt/pysetup" \
    VENV_PATH="/opt/pysetup/.venv" \
    CC=gcc

# prepend poetry and venv to path
ENV PATH="$POETRY_HOME/bin:$VENV_PATH/bin:$PATH"

# Add needed packages to run Mealie (eg usermod)
RUN apk add build-base=0.5-r2 \
    shadow=4.8.1-r0 \
    caddy=2.3.0-r0

# Add poetry build deps
RUN apk add freetype-dev=2.10.4-r1 \
    fribidi-dev=1.0.10-r0 \
    harfbuzz-dev=2.7.4-r1 \
    jpeg-dev=9d-r1 \
    lcms2-dev=2.11-r0 \
    openjpeg-dev=2.4.0-r1 \
    tcl-dev=8.6.10-r1 \
    tiff-dev=4.2.0-r0 \
    tk-dev=8.6.10-r1 \
    zlib-dev=1.2.11-r3 \
    musl-dev=1.2.2-r1 \
    libxml2-dev=2.9.10-r7 \
    libxslt-dev=1.1.34-r0 \
    gcc=10.2.1_pre1-r3 	 \
    python3-dev=3.8.10-r0 \
    libffi-dev=3.3-r2 \
    openssl-dev=1.1.1k-r0 \
    cargo=1.47.0-r2 \
    tzdata=2021a-r0
    # TODO: Configure TZ


RUN addgroup -g 911 abc \
    && adduser -u 911 -h $MEALIE_HOME -s /bin/bash -G abc -D abc \
    && mkdir -p $MEALIE_HOME

# Mealie Production image
ENV PRODUCTION=true


# copying poetry and venv into image
COPY --from=mealie $POETRY_HOME $POETRY_HOME
COPY --from=mealie $PYSETUP_PATH $PYSETUP_PATH

# copy backend
COPY --from=mealie $MEALIE_HOME $MEALIE_HOME

WORKDIR $MEALIE_HOME
RUN . $VENV_PATH/bin/activate \
    && pip uninstall PIL \
    && $POETRY_HOME/bin/poetry install \
    # Reinstall pillow and uvloop, has system deps
    && pip install --force-reinstall pillow \
    && pip install --force-reinstall uvloop

WORKDIR /

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
    maintainer="Tom Matheussen <tommatheussen5@hotmail.com>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Tom's Home Assistant Add-ons" \
    org.opencontainers.image.authors="Tom Matheussen <tommatheussen5@hotmail.com>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://github.com/Tommatheussen/HomeAssistant-Addons" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
