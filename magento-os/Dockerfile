FROM ubuntu:18.04
LABEL maintainer="bill@halfpricebanners.com"

# We make fairly extensive use of ARGs here so that the history of the docker file
# doesn't remember them, and also so that they can be overridden on the CLI if you
# so desire. Honestly, since you're running this container locally, the defaults
# (including credentials) should be fine.  However, if you want to change things,
# this ARGs section is where you should do it.

# Set Up Timezone - See https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
# for valid TZ strings
ARG TZ=America/Chicago
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# MySQL database variables
ARG MDB=magento
ARG MDBUSER=magento
ARG MDBPASS=mag3DeeBee*

# Magento admin credentials
ARG MAFNAME=Galileo
ARG MALNAME=Galilei
ARG MAURL=admin
ARG MAUSER=admin
ARG MAPASS=mageU123
ARG MAEMAIL=therealgalileo1564@gmail.com

# Going forward, no end-user-defined variables exist. 
#
# Here there be dragons.

# Tell the system that there's no one that can tapitty-type
ARG DEBIAN_FRONTEND=noninteractive

# Install Necessary Packages

# System
RUN apt update
RUN apt install -y lsof
RUN apt install -y wget
RUN apt install -y sudo
RUN apt install -y vim
RUN apt install -y git

# Services + PHP Mods
RUN apt install -y sendmail
RUN apt install -y mysql-server-5.7
RUN apt install -y nginx
RUN apt install -y php php-bcmath php-curl php-gd php-intl php-mbstring php-mysql php-soap php-xdebug php-xml php-zip
RUN apt install -y php-fpm
RUN apt install -y composer
RUN wget -O phpunit https://phar.phpunit.de/phpunit-8.phar && chmod +x phpunit && mv phpunit /usr/bin/

# Add User, grant sudo permissions
RUN useradd -m -s /bin/bash magento
RUN usermod -G sudo -a magento

# Copy in relevant config files
COPY ./etc/nginx/sites-available/magento-nginx.conf /etc/nginx/sites-available/
COPY ./etc/php/7.2/fpm/php.ini /etc/php/7.2/fpm/
COPY ./etc/php/7.2/fpm/pool.d/magento-fpm.conf /etc/php/7.2/fpm/pool.d/
COPY ./etc/php/7.2/mods-available/xdebug.ini /etc/php/7.2/mods-available/
COPY ./etc/sudoers /etc/

# Remove remnant default files, link up new config
RUN rm -f /etc/php/7.2/fpm/pool.d/www.conf
RUN rm -f /etc/nginx/sites-enabled/default
RUN ln -s /etc/nginx/sites-available/magento-nginx.conf /etc/nginx/sites-enabled/magento-nginx.conf

# Prep SQL service for Magento DB
RUN chown -R mysql:mysql /var/lib/mysql && /etc/init.d/mysql start && \
	mysql --defaults-file=/etc/mysql/debian.cnf \
    -e "CREATE DATABASE ${MDB}; GRANT ALL ON ${MDB}.* TO '${MDBUSER}'@'localhost' IDENTIFIED BY '${MDBPASS}'"

# System work is finished. Let's get magento up and running
USER magento
WORKDIR /home/magento
RUN git clone https://github.com/magento/magento2.git www 
WORKDIR /home/magento/www
RUN git checkout 2.3.1 --force && \
	git checkout -b magento_2_3_1 && \
	rm -rf .git && \
	git init && \
	git remote add origin https://github.com/magento/magento2-sample-data.git && \
	git fetch && \
	git checkout 2.3.1 --force && \
	git pull origin 2.3.1 && \
	rm -rf .git && \
	composer install
RUN sudo chown -R mysql:mysql /var/lib/mysql && \
	sudo service mysql start && \
	sudo service sendmail start && \
	sudo service php7.2-fpm start && \
	sudo service nginx start && \
	php bin/magento setup:install --base-url=http://magento2u.loc:8088 \
	--admin-firstname=${MAFNAME} --admin-lastname=${MALNAME} --backend-frontname=${MAURL} \
	--admin-email=${MAEMAIL} --admin-user=${MAUSER} --admin-password=${MAPASS} \
	--db-name=${MDB} --db-host=localhost --db-user=${MDBUSER} --db-password=${MDBPASS} \
	--use-rewrites=1 --admin-use-security-key=0

# Final tidbits
RUN sudo apt --purge autoremove -y ; sudo apt clean

# ENTRYPOINT [ "bash" ]

ENTRYPOINT sudo chown -R mysql:mysql /var/lib/mysql && \
	sudo service mysql start && \
	sudo service sendmail start && \
	sudo service php7.2-fpm start && \
	sudo service nginx start && bash

EXPOSE 80