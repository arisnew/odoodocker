# Odoo18-fastapi Dockerfile
# Deskripsi: Dockerfile untuk menjalankan Odoo 18 dengan FastAPI di dalam container Docker
# NOTE: Odoo dijalankan di dalam virtual environment Python untuk isolasi dependensi

FROM ubuntu:noble

LABEL maintainer="aris.new2011@gmail.com"

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG=en_US.UTF-8

# Retrieve the target architecture to install the correct wkhtmltopdf package
ARG TARGETARCH

# Variabel lingkungan
ENV ODOO_VERSION=18.0 \
    VENV_PATH=/opt/odoo/venv \
    ODOO_HOME=/opt/odoo \
    ADDONS_PATH=/mnt/extra-addons \
    ENTERPRISE_PATH=/mnt/enterprise-addons \
    DATA_PATH=/var/lib/odoo \
    CONFIG_PATH=/etc/odoo

# Install dependencies, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive \
    apt-get install -y \
        python3 \
        python3-venv \
        python3-pip \
        git \
        wget \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        node-less \
        libpq-dev \
        build-essential \
        libxml2-dev \
        libxslt1-dev \
        libsasl2-dev \
        libldap2-dev \
        libssl-dev \
        libjpeg-dev \
        libpng-dev \
        libffi-dev \
        npm \
        xz-utils && \
    if [ -z "${TARGETARCH}" ]; then \
        TARGETARCH="$(dpkg --print-architecture)"; \
    fi; \
    WKHTMLTOPDF_ARCH=${TARGETARCH} && \
    case ${TARGETARCH} in \
    "amd64") WKHTMLTOPDF_ARCH=amd64 && WKHTMLTOPDF_SHA=967390a759707337b46d1c02452e2bb6b2dc6d59  ;; \
    "arm64")  WKHTMLTOPDF_SHA=90f6e69896d51ef77339d3f3a20f8582bdf496cc  ;; \
    "ppc64le" | "ppc64el") WKHTMLTOPDF_ARCH=ppc64el && WKHTMLTOPDF_SHA=5312d7d34a25b321282929df82e3574319aed25c  ;; \
    esac \
    && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.jammy_${WKHTMLTOPDF_ARCH}.deb \
    && echo ${WKHTMLTOPDF_SHA} wkhtmltox.deb | sha1sum -c - \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ noble-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update  \
    && apt-get install --no-install-recommends -y postgresql-client \
    && rm -f /etc/apt/sources.list.d/pgdg.list \
    && rm -rf /var/lib/apt/lists/*

# install libreoffice
RUN apt-get update && apt-get install -y --no-install-recommends libreoffice

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

# make sure the directories exist
RUN mkdir -p $ODOO_HOME $ADDONS_PATH $ENTERPRISE_PATH $DATA_PATH $CONFIG_PATH

# Clone Odoo repository
RUN git clone --depth 1 --branch $ODOO_VERSION https://github.com/odoo/odoo.git  $ODOO_HOME

# Buat dan aktifkan virtual environment
RUN python3 -m venv $VENV_PATH

# Upgrade pip di dalam virtual environment
RUN $VENV_PATH/bin/pip install --upgrade pip

# Instal dependensi Odoo menggunakan requirements.txt di dalam virtual environment
RUN $VENV_PATH/bin/pip install -r $ODOO_HOME/requirements.txt

# Instal FastAPI versi terbaru dan lib lain di dalam virtual environment
RUN $VENV_PATH/bin/pip install fastapi uvicorn packaging python-multipart pyjwt a2wsgi ujson parse-accept-language

# Install py3o.template py3o.formats
RUN $VENV_PATH/bin/pip install py3o.template py3o.formats

# Install pyfcm sentry
RUN $VENV_PATH/bin/pip install pyfcm sentry-sdk==2.22.0

# Copy template konfigurasi Odoo
COPY odoo.conf $CONFIG_PATH/odoo.conf

VOLUME ["/var/lib/odoo", "/mnt/extra-addons", "/mnt/enterprise-addons"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# NOTE tidak menggunakan entrypoint.sh, karena Odoo dijalankan langsung dari virtual environment

# Jalankan Odoo menggunakan virtual environment
CMD ["/bin/bash", "-c", "source $VENV_PATH/bin/activate && python /opt/odoo/odoo-bin -c $CONFIG_PATH/odoo.conf"]