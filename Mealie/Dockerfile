ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base-debian:stretch
# ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base-python:3.9-alpine3.13
###############################################################################
# Get prebuild containers from Mealie
###############################################################################
FROM "hkotel/mealie:dev" as mealie

FROM $BUILD_FROM

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# COPY --from=mealie /app /app
# COPY --from=mealie /opt/pysetup /opt/pysetup
# COPY --from=mealie /opt/poetry /opt/poetry

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
    VENV_PATH="/opt/pysetup/.venv"

# prepend poetry and venv to path
ENV PATH="$POETRY_HOME/bin:$VENV_PATH/bin:$PATH"


RUN apt update && apt install -y wget build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev

RUN wget https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tgz
RUN tar -xf Python-3.9.1.tgz

WORKDIR ./Python-3.9.1
RUN ./configure --enable-optimizations
RUN make -j 2

RUN make altinstall

RUN ln -s /usr/local/bin/python3.9 /usr/local/bin/python
RUN ln -s /usr/local/bin/python3.9 /usr/local/bin/python3

# RUN which python3.9
# RUN which python3
# RUN which python

RUN  apt install -y debian-keyring debian-archive-keyring apt-transport-https \
 && curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | apt-key add - \
    && curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list \
     && apt-get update \
    && apt-get install --no-install-recommends -y \
    caddy

RUN mkdir /mem

# RUN apk --no-cache add shadow \
#         pcre-dev \
#         libc-dev \
#         linux-headers \
#         python3-dev \
#         nginx \
#         curl \
#         bash \
#         gcc \
#         g++ \
#         uwsgi-python3 \
#         # psycopg
#         postgresql-dev \
#         musl-dev \
#         libffi-dev \
#         freetype-dev \
#         lcms2-dev \
#         # PILLOW
#         jpeg-dev \
#         zlib-dev \
#         # pylibmc
#         libmemcached-dev \
#         openjpeg-dev \
#         jpeg \
#         libjpeg \
#         openjpeg \
#         tiff-dev \
#         tk-dev \
#         tcl-dev \
#         harfbuzz-dev \
#         fribidi-dev \
#         # opencv
#         make \
#         cmake \
#         clang \
#         clang-dev \
#         ninja \
#         # tesseract
#         leptonica-dev \
#         tesseract-ocr \
#         tesseract-ocr-dev \
#         openssh-server \
#         gcc \
#         musl-dev \
#         zlib-dev \
#         jpeg-dev


# RUN addgroup -g 911 abc && adduser -u 911 -h $MEALIE_HOME -s /bin/bash -G abc && mkdir $MEALIE_HOME

RUN useradd -u 911 -U -d $MEALIE_HOME -s /bin/bash abc \
    && usermod -G users abc \
    && mkdir $MEALIE_HOME

# create user account
# RUN adduser -u 911 -D -h $MEALIE_HOME -s /bin/bash abc \
#     && addgroup abc users


# ENV POETRY_VERSION=1.1.6
# RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/install-poetry.py | python -

# Mealie Production image
ENV PRODUCTION=true

# ENV DB_URL=/data/mealie.db
# RUN echo "$PATH"

# RUN poetry -V

# curl for used by healthcheck
# RUN apt-get update \
#     && apt-get install --no-install-recommends -y \
#     curl \
#     && apt-get autoremove \
#     && rm -rf /var/lib/apt/lists/*


# copying poetry and venv into image
COPY --from=mealie $POETRY_HOME $POETRY_HOME
COPY --from=mealie $PYSETUP_PATH $PYSETUP_PATH

# copying caddy into image
# COPY -- /usr/bin/caddy /usr/bin/caddy

# copy backend
COPY --from=mealie $MEALIE_HOME $MEALIE_HOME

WORKDIR $MEALIE_HOME
RUN . $VENV_PATH/bin/activate && $POETRY_HOME/bin/poetry install
WORKDIR /


# RUN ln /app/data /data
# RUN ln -s /app/data/mealie_v0.5.0.db /data/mealie.db

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



# VOLUME [ "/app/data/" ]

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
