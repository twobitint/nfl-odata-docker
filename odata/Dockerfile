FROM centos:7

# Install PHP7 RPM (Webtatic)
RUN curl -O https://mirror.webtatic.com/yum/RPM-GPG-KEY-webtatic-el7 \
    && rpm --import ./RPM-GPG-KEY-webtatic-el7 \
    && rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
    && rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

# Install the PHP and Apache services
RUN yum -y update && yum -y install \
    php70w \
    php70w-opcache \
    php70w-common \
    php70w-pdo \
    php70w-pgsql \
    httpd \
    mod_ssl \
    openssl \
    && yum clean all \
    && rm /etc/httpd/conf.d/ssl.conf

# Set up config files
COPY ./vhost.conf /etc/httpd/conf.d/vhost.conf

# Create self-signed cert for SSL
RUN mkdir /etc/httpd/ssl \
    && openssl req \
    -new \
    -newkey rsa:4096 \
    -days 365 \
    -nodes \
    -x509 \
    -subj "/C=US/ST=Denial/L=Springfield/O=Dis/CN=domain-one.com" \
    -keyout /etc/httpd/ssl/apache.key \
    -out /etc/httpd/ssl/apache.crt

COPY get-composer.sh /

# Need Laravel dependencies
RUN yum install -y wget git \
    && chmod 775 get-composer.sh \
    && ./get-composer.sh \
    && rm -r /var/www/html \
    && git clone https://github.com/twobitint/nfl-odata.git /var/www/html \
    && cd /var/www/html \
    && chown -R apache storage \
    && chown -R apache bootstrap/cache \
    && yum install -y php70w-mbstring php70w-xml \
    && php /composer.phar install

RUN cd /var/www/html \
    && cp .env.example .env \
    && php artisan key:generate

# Start Apache
EXPOSE 80 443
ADD run-httpd.sh /run-httpd.sh
RUN chmod -v +x /run-httpd.sh
CMD ["/run-httpd.sh"]