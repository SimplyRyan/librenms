FROM php:7.3-apache-buster
MAINTAINER iffanux <iffanux@gmail.com>
LABEL Description="LibreNMS - APACHE2"

USER root
RUN rm -rf /etc/localtime
RUN ln -s /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
RUN apt  update
RUN apt  upgrade
RUN apt install -y zlib1g-dev libfreetype6-dev libjpeg62-turbo-dev libzip-dev libsnmp-dev mtr-tiny nmap nagios-plugins ipmitool nano 
RUN apt install -y git fping graphviz imagemagick acl python-memcache python-mysqldb rrdtool snmp snmpd whois rrdcached unzip
RUN docker-php-ext-configure gd
RUN docker-php-ext-install -j$(nproc) pdo gd
RUN docker-php-ext-install zip snmp pdo_mysql sockets
RUN docker-php-ext-enable zip snmp pdo_mysql sockets gd
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer

RUN useradd librenms -d /opt/librenms -M -r
RUN usermod -a -G librenms www-data
RUN git clone https://github.com/librenms/librenms.git /opt/librenms
RUN chown -R librenms:librenms /opt/librenms/
RUN chmod 770 /opt/librenms
RUN chmod 775 /opt/librenms/rrd
RUN chmod 777 /opt/librenms/logs -R
RUN setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
RUN setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/

USER librenms
RUN  cd /opt/librenms && ./scripts/composer_wrapper.php install --no-dev
#RUN ./opt/librenms/scripts/composer_wrapper.php install --no-dev

USER root
RUN mkdir /data
RUN mv /opt/librenms/rrd /data/
RUN ln -s /data/rrd /opt/librenms/
RUN mv /opt/librenms/logs /data/
RUN ln -s /data/logs /opt/librenms/
RUN chown -R librenms:librenms /opt/librenms/
RUN setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
RUN setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/

RUN curl -o /usr/bin/distro https://raw.githubusercontent.com/librenms/librenms-agent/master/snmp/distro
RUN chmod +x /usr/bin/distro
COPY librenms.conf /etc/apache2/sites-available/000-default.conf
RUN a2enmod rewrite
RUN a2enmod headers
RUN cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
RUN mkdir /etc/cron.d/
RUN cp /opt/librenms/librenms.nonroot.cron /etc/cron.d/librenms
RUN cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms

# RRDCached
#EXPOSE 42217/tcp
# SysLog
#EXPOSE 514/tcp
#EXPOSE 514/udp

RUN apt-get clean

VOLUME /data
