FROM debian:bookworm
ARG DOCKER_ODOO_UID=1000  
ARG DOCKER_ODOO_GID=1000
ARG PG_MAJOR=15

SHELL ["/bin/bash", "-euxo", "pipefail", "-c"]

ENV LANG=C.UTF-8

ARG TARGETARCH

RUN set -eux; \
    apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl gnupg dirmngr \
    && echo 'deb http://apt.postgresql.org/pub/repos/apt/ bookworm-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && curl -fsSL https://deb.nodesource.com/setup_18.x | bash -

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    bash-completion \
    build-essential \
    ca-certificates \
    cargo \
    curl \
    fonts-crosextra-carlito \
    fonts-noto-cjk \
    freetds-dev \
    git \
    libbz2-dev \
    libc6-dev \
    libcups2-dev \
    libevent-dev \
    libffi-dev \
    libfontconfig \
    libfontconfig1-dev \
    libfreetype6-dev \
    libfribidi-dev \
    libharfbuzz-dev \
    libjpeg-dev \
    libjpeg62-turbo-dev \
    liblcms2-dev \
    libkrb5-dev \
    libldap2-dev \
    libmagic1 \
    libopenjp2-7-dev \
    libpq-dev \
    libsasl2-dev \
    libssl-dev \
    libtiff5-dev \
    libwebp-dev \
    libx11-dev \
    libxcb1-dev \
    libxml2 \
    libxml2-dev \
    libxrender-dev \
    libxslt1-dev \
    libxtst-dev \
    locales \
    nodejs \
    openssh-client \
    poppler-utils \
    postgresql-client-${PG_MAJOR} \
    python3-dev \
    python3-magic \
    python3-num2words \
    python3-odf \
    python3-pdfminer \
    python3-phonenumbers \
    python3-pip \
    python3-pyldap \
    python3-qrcode \
    python3-renderpm \
    python3-setuptools \
    python3-slugify \
    python3-venv \
    python3-vobject \
    python3-watchdog \
    python3-xlrd \
    python3-xlwt \
    sudo \
    unixodbc \
    wget \
    xvfb \
    xz-utils \
    zlib1g \
    zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*


RUN set -eux; \
    : "${TARGETARCH:=$(dpkg --print-architecture)}"; \
    case "${TARGETARCH}" in \
    amd64)   WKHTMLTOPDF_ARCH="amd64";   WKHTMLTOPDF_SHA="e9f95436298c77cc9406bd4bbd242f4771d0a4b2" ;; \
    arm64)   WKHTMLTOPDF_ARCH="arm64";   WKHTMLTOPDF_SHA="77bc06be5e543510140e6728e11b7c22504080d4" ;; \
    ppc64le|ppc64el) WKHTMLTOPDF_ARCH="ppc64el"; WKHTMLTOPDF_SHA="d61c2497fa6edb4650548b8864f53c7de161347d" ;; \
    *) echo "Unsupported TARGETARCH=${TARGETARCH}"; exit 1 ;; \
    esac; \
    curl -fsSL -o /tmp/wkhtmltox.deb "https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.bookworm_${WKHTMLTOPDF_ARCH}.deb"; \
    echo "${WKHTMLTOPDF_SHA}  /tmp/wkhtmltox.deb" | sha1sum -c -; \
    apt-get update; apt-get install -y --no-install-recommends /tmp/wkhtmltox.deb; \
    rm -rf /var/lib/apt/lists/* /tmp/wkhtmltox.deb

# Создаем пользователя odoo
RUN groupadd --gid ${DOCKER_ODOO_GID} odoo \
    && useradd odoo -u ${DOCKER_ODOO_UID} -g ${DOCKER_ODOO_GID} -m -s /bin/bash \
    && usermod -aG sudo odoo \
    && echo "odoo ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/odoo \
    && chmod 0440 /etc/sudoers.d/odoo \
    && echo "export PATH=\"\$PATH:/home/odoo/.local/bin\"" >> /home/odoo/.bashrc

RUN mkdir -p /workspace/extra_addons \
    /workspace/conf \
    /workspace/odoo \
    /opt/odoo \
    && chown -R odoo:odoo /workspace /opt/odoo

WORKDIR /usr/local/lib

RUN npm install -g npm@10 less \
    && npm install --global prettier @prettier/plugin-xml prettier-plugin-sql \
    && npm cache clean --force

COPY conf/odoo-server.conf /workspace/conf/odoo-server.conf

USER odoo
ENV VIRTUAL_ENV=/opt/odoo/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
WORKDIR /workspace

# https://github.com/gevent/gevent/issues/2076 - install specific gevent version to avoid build errors
RUN "$VIRTUAL_ENV/bin/python" -m pip install --upgrade pip wheel debugpy pre-commit \
    && "$VIRTUAL_ENV/bin/python" -m pip install -r https://raw.githubusercontent.com/odoo/odoo/refs/heads/16.0/requirements.txt

# Expose Odoo services
EXPOSE 8069 8071 8072

ENV ODOO_RC=/workspace/conf/odoo-server.conf