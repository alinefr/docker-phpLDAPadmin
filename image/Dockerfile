FROM alinefr/web-baseimage:2.0.0

# phpLDAPadmin version
ARG PHPLDAPADMIN_VERSION=1.2.6.7

# Add multiple process stack to supervise apache2 and php7.3-fpm
# sources: https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/add-multiple-process-stack
#          https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/add-service-available
#          https://github.com/osixia/docker-web-baseimage/blob/stable/image/service-available/:apache2/download.sh
#          https://github.com/osixia/docker-web-baseimage/blob/stable/image/service-available/:php7.3-fpm/download.sh
#          https://github.com/osixia/light-baseimage/blob/stable/image/service-available/:ssl-tools/download.sh
# Install ca-certificates, curl and php dependencies
# Download phpLDAPadmin, check file integrity, and unzip phpLDAPadmin to /var/www/phpldapadmin_bootstrap
# Remove curl
RUN apt-get update \
  && /container/tool/add-multiple-process-stack \
  && /container/tool/add-service-available :apache2 :php8.2-fpm :ssl-tools \
  && LC_ALL=C DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  ca-certificates \
  curl \
  libldap-common \
  php8.2-ldap \
  php8.2-readline \
  php8.2-xml \
  && curl -o phpldapadmin.tar.gz -SL https://github.com/leenooks/phpLDAPadmin/archive/${PHPLDAPADMIN_VERSION}.tar.gz \
  && mkdir -p /var/www/phpldapadmin_bootstrap /var/www/phpldapadmin \
  && tar -xzf phpldapadmin.tar.gz --strip 1 -C /var/www/phpldapadmin_bootstrap \
  && apt-get remove -y --purge --auto-remove curl ca-certificates \
  && rm phpldapadmin.tar.gz \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Workaround to make PHPLdapAdmin to work on PHP 8.2 before it gets properly fixed upstream.
RUN sed -i '/class\ page/i#[\AllowDynamicProperties]' /var/www/phpldapadmin_bootstrap/lib/page.php \
  && sed -i '/class\ Template/i#[\AllowDynamicProperties]' /var/www/phpldapadmin_bootstrap/lib/Template.php \
  && sed -i '/class\ TemplateRender/i#[\AllowDynamicProperties]' /var/www/phpldapadmin_bootstrap/lib/TemplateRender.php \
  && sed -i '/class\ Query/i#[\AllowDynamicProperties]' /var/www/phpldapadmin_bootstrap/lib/Query.php \
  && rm /etc/apache2/sites-enabled/000-default.conf

# Add service directory to /container/service
ADD service /container/service

# Use baseimage install-service script
# https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/install-service
RUN /container/tool/install-service

# Add default env variables
ADD environment /container/environment/99-default

# Set phpLDAPadmin data directory in a data volume
VOLUME ["/var/www/phpldapadmin"]

# Expose http and https default ports
EXPOSE 80 443
