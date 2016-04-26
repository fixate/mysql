FROM debian:jessie

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN mkdir /docker-entrypoint-initdb.d

# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
RUN apt-get update && apt-get install -y perl --no-install-recommends && rm -rf /var/lib/apt/lists/*

# mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
RUN apt-get update && apt-get install -y libaio1 pwgen && rm -rf /var/lib/apt/lists/*

COPY mysql-server_5.0.90-2_amd64.deb /tmp
COPY mysql-client_5.0.90-2_amd64.deb /tmp
COPY libmysqlclient.so.15 /usr/lib

RUN dpkg -i /tmp/mysql-server_5.0.90-2_amd64.deb \
  && rm /tmp/mysql-server_5.0.90-2_amd64.deb \
  && dpkg -i /tmp/mysql-client_5.0.90-2_amd64.deb \
  && rm /tmp/mysql-client_5.0.90-2_amd64.deb

ENV PATH $PATH:/usr/local/mysql/bin:/usr/local/mysql/scripts

# replicate some of the way the APT package configuration works
# this is only for 5.5 since it doesn't have an APT repo, and will go away when 5.5 does
RUN mkdir -p /etc/mysql/conf.d \
	&& { \
		echo '[mysqld]'; \
		echo 'skip-host-cache'; \
		echo 'skip-name-resolve'; \
		echo 'user = mysql'; \
		echo 'datadir = /var/lib/mysql'; \
		echo '!includedir /etc/mysql/conf.d/'; \
	} > /etc/mysql/my.cnf

VOLUME /var/lib/mysql

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306
CMD ["mysqld", "--user=mysql"]
