FROM python:3.8.8-slim-buster AS base

ARG BOT_PROJECT="captcha-bot"
ARG BOT_USER="nobody"
ARG BOT_GROUP="nogroup"
ARG BOT_HOME_DIR="/srv"
ARG APP_DIR="${BOT_HOME_DIR}/app"
ARG GITHUB_URL="https://github.com/smartfon2021/Telegram_captcha"

ENV BOT_PROJECT="${BOT_PROJECT}" \
    BOT_USER="${BOT_USER}" \
    BOT_GROUP="${BOT_GROUP}" \
    BOT_HOME_DIR="${BOT_HOME_DIR}" \
    APP_DIR="${APP_DIR}" \
    GITHUB_URL="${GITHUB_URL}" \
    DEBIAN_FRONTEND=noninteractive \
    APT_OPTS="-q=2 --no-install-recommends --yes"

RUN chown -cR "${BOT_USER}:${BOT_GROUP}" ${BOT_HOME_DIR} && \
    usermod -d ${BOT_HOME_DIR} ${BOT_USER}


FROM base AS builder-deps

# Install build dependencies
RUN apt-get ${APT_OPTS} update && \
    apt-get ${APT_OPTS} install \
    build-essential \
    git \
    procps  \
    libtiff5-dev \
    libjpeg62-turbo-dev \
    zlib1g-dev \
    libfreetype6-dev \
    liblcms2-dev \
    libwebp-dev \
    tcl8.6-dev \
    tk8.6-dev \
    python3-tk


FROM builder-deps AS builder

USER ${BOT_USER}
WORKDIR ${BOT_HOME_DIR}
RUN git clone ${GITHUB_URL} ${APP_DIR} && \
    python3 -m pip install --user --requirement ${APP_DIR}/requirements.txt && \
    cd ${APP_DIR}/src && \
    chown -cR ${BOT_USER}:${BOT_GROUP} ${BOT_HOME_DIR} && \
    rm -rf ${BOT_HOME_DIR}/.cache && \
    find ${APP_DIR} -iname '.git*' -print0 | xargs -0 -r -t rm -rf


FROM base AS app

ENV PATH="${PATH}:${BOT_HOME_DIR}/.local/bin"

COPY --from=builder ${BOT_HOME_DIR} ${BOT_HOME_DIR}

RUN chown -R "${BOT_USER}:${BOT_GROUP}" ${BOT_HOME_DIR} && \
    usermod -d ${BOT_HOME_DIR} ${BOT_USER}

USER ${BOT_USER}
WORKDIR ${APP_DIR}
CMD ["./tools/entrypoint.sh"]
