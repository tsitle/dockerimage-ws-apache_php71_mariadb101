ARG CF_APACHE_BASE_IMGFULL

FROM ${CF_APACHE_BASE_IMGFULL}

ARG CF_APACHE_BASE_VER

ENV DEBIAN_FRONTEND=noninteractive

ARG CF_PHP_FPM_VERSION="7.1"
ENV CF_PHP_FPM_VERSION=$CF_PHP_FPM_VERSION

RUN \
	apt-get update \
	&& apt-get upgrade -y \
	&& apt-get dist-upgrade -y \
	# dotdeb sources
	&& echo "deb https://packages.sury.org/php/ stretch main" >> /etc/apt/sources.list.d/php.list \
	# additional dotdeb source keys
	&& wget -q https://packages.sury.org/php/apt.gpg -O- | apt-key add - \
	# update package lists
	&& apt-get update

RUN \
	# install packages
		apt-get update \
		&& apt-get install -y --no-install-recommends \
			# mariadb packages
				mariadb-client \
			# php packages
				php${CF_PHP_FPM_VERSION} \
				php${CF_PHP_FPM_VERSION}-fpm \
				php${CF_PHP_FPM_VERSION}-cli \
				php${CF_PHP_FPM_VERSION}-common \
				php${CF_PHP_FPM_VERSION}-opcache \
				php${CF_PHP_FPM_VERSION}-curl \
				php${CF_PHP_FPM_VERSION}-gd \
				php${CF_PHP_FPM_VERSION}-imagick \
				php${CF_PHP_FPM_VERSION}-imap \
				php${CF_PHP_FPM_VERSION}-intl \
				php${CF_PHP_FPM_VERSION}-json \
				php${CF_PHP_FPM_VERSION}-mbstring \
				php${CF_PHP_FPM_VERSION}-mysql \
				php${CF_PHP_FPM_VERSION}-readline \
				php${CF_PHP_FPM_VERSION}-sqlite3 \
				php${CF_PHP_FPM_VERSION}-zip \
				php${CF_PHP_FPM_VERSION}-xml \
				php${CF_PHP_FPM_VERSION}-xdebug \
				php${CF_PHP_FPM_VERSION}-bcmath \
				php${CF_PHP_FPM_VERSION}-mcrypt \
				#libapache2-mod-php${CF_PHP_FPM_VERSION} \
				php-pear \
				xml-core \
			# other packages
				composer

#
RUN \
	apt-get --quiet --yes autoclean \
	&& apt-get --quiet --yes autoremove \
	&& apt-get --quiet --yes clean \
	&& rm -rf \
			/usr/share/man \
			/usr/share/doc \
			/usr/share/icons \
			/usr/share/poppler \
			/usr/share/mime \
			/var/lib/apt/lists*

#
ENV DEBIAN_FRONTEND=dialog

#
RUN \
	( a2dismod php${CF_PHP_FPM_VERSION} 2>/dev/null || echo -n "" ) \
	&& a2enconf php${CF_PHP_FPM_VERSION}-fpm \
	&& phpdismod xdebug

# copy config files
COPY files/php/ /etc/php/${CF_PHP_FPM_VERSION}/
COPY files/apache/000-default.conf /etc/apache2/sites-available/

#
ENV TMP_WEBROOT_LAST_SITE=$CF_WEBROOT_SITE
ENV CF_WEBROOT_SITE=site-php

RUN \
	[ -n "${TMP_WEBROOT_LAST_SITE}" -a "${TMP_WEBROOT_LAST_SITE}" != "." -a "${TMP_WEBROOT_LAST_SITE}" != "./" ] \
	&& mv ${CF_WEBROOT}/${TMP_WEBROOT_LAST_SITE}/index.html ${CF_WEBROOT}/ \
	&& rmdir ${CF_WEBROOT}/${TMP_WEBROOT_LAST_SITE} \
	&& mkdir ${CF_WEBROOT}/${CF_WEBROOT_SITE} \
	&& chmod u=rwx,g=rwxs,o= ${CF_WEBROOT}/${CF_WEBROOT_SITE} \
	&& mv ${CF_WEBROOT}/index.html ${CF_WEBROOT}/${CF_WEBROOT_SITE}/

ENV TMP_WEBROOT_LAST_SITE=
