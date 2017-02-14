FROM debian:jessie
MAINTAINER Odoo S.A. <info@odoo.com>

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends --fix-missing \
            ca-certificates \
            curl \
            wget \
            node-less \
            node-clean-css \
            python-pyinotify \
            python-renderpm \
            python-support \
            ipython \
            python-xlrd \
            python-pip \
            python-gevent \
            python-ofxparse \
            ghostscript \
            libgraphicsmagick1-dev \
            libmagickwand-6.q16-dev \
        && curl -o wkhtmltox.deb -SL http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb \
        && echo '40e8b906de658a2221b15e4e8cd82565a47d7ee8 wkhtmltox.deb' | sha1sum -c - \
        && dpkg --force-depends -i wkhtmltox.deb \
        && apt-get -y install -f --no-install-recommends \
        && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Install PIP dependencies
# EbaySDK requirements
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends --fix-missing \
            libxslt1-dev \
            python-dev \
            gcc \
        && rm -rf /var/lib/apt/lists/*

RUN set -x; \
    pip install -U pip

RUN set -x; \
    pip install -U raven Wand
    
RUN set -x; \
    pip install -U ebaysdk

# Install Odoo
ENV ODOO_VERSION 9.0
ENV ODOO_RELEASE 20170213
ENV RELEASE_CHECKSUM 07bfc697ef9de90261ff6772087bc5951b1bb234
RUN set -x; \
        curl -o odoo.deb -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}c.${ODOO_RELEASE}_all.deb \
        && echo "${RELEASE_CHECKSUM} odoo.deb" | sha1sum -c - \
        && dpkg --force-depends -i odoo.deb \
        && apt-get update \
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* odoo.deb

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./openerp-server.conf /etc/odoo/
RUN chown odoo /etc/odoo/openerp-server.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
        && chown -R odoo /mnt/extra-addons \
        && mkdir -p /mnt/custom-addons \
        && chown -R odoo /mnt/custom-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons", "/mnt/custom-addons"]

# For PostgreSQL dump in container
# Do it for 9.5, Jessie by default has 9.4
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ jessie-pgdg main' > /etc/apt/sources.list.d/pgdg.list
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
RUN set -x; \
    apt-get update \
    && apt-get install -y postgresql-client-9.6

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV OPENERP_SERVER /etc/odoo/openerp-server.conf

# gevent stuff
RUN set -x; \
    pip install -U psycogreen

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["openerp-server"]
