# zabbix-postgresql11-php71

#### Start<br/>
`yum clean all`<br/>
`yum -y update`<br/>

#### nano & net-tools<br/>
`yum -y install nano`<br/>
`yum -y install net-tools`<br/>

#### off firewalld <br/>
`systemctl stop firewalld` <br/>
`systemctl disable firewalld` <br/>

#### off selinux <br/>
`nano /etc/sysconfig/selinux` <br/>
	SELINUX=disabled<br/>
`setenforce 0`<br/>
`reboot`<br/>

#apache
yum install -y httpd
systemctl enable httpd
systemctl start httpd
netstat -tulnp | grep httpd

#php71
yum install epel-release
rpm -Uhv http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum install yum-utils
yum-config-manager --enable remi-php71
yum install php71 php-fpm php-cli php-mysql php-gd php-ldap php-odbc php-pdo php-pecl-memcache php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap php-bcmath
systemctl start php-fpm
systemctl enable php-fpm
netstat -tulpn | grep php-fpm

#postgresql11
yum install https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/pgdg-centos11-11-2.noarch.rpm
yum install postgresql11 postgresql11-server
/usr/pgsql-11/bin/postgresql-11-setup initdb
systemctl enable postgresql-11
systemctl start postgresql-11
nano /var/lib/pgsql/11/data/pg_hba.conf
	###change
	# "local" is for Unix domain socket connections only
	local   all             all                                     peer
	# IPv4 local connections:
	host    all             all             127.0.0.1/32            ident
	# IPv6 local connections:
	host    all             all             ::1/128                 ident
	###By:
	# "local" is for Unix domain socket connections only
	local   all             all                                     trust
	# IPv4 local connections:
	host    all             all             127.0.0.1/32            trust
	# IPv6 local connections:
	host    all             all             ::1/128                 trust
systemctl reload postgresql-11

#create user and base zabbix postgresql
su - postgres
createuser zabbix -P
createdb -O zabbix zabbix
exit

#install zabbix
rpm -i https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
yum install zabbix-server-pgsql zabbix-web-pgsql zabbix-agent
zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | sudo -u zabbix psql zabbix
nano /etc/zabbix/zabbix_server.conf
	DBHost=localhost
	DBName=zabbix
	DBUser=zabbix
	DBPassword=zabbix
nano /etc/httpd/conf.d/zabbix.conf
	#change <IfModule mod_php5.c> to <IfModule mod_php7.c>
	php_value date.timezone Asia/Almaty
systemctl restart zabbix-server zabbix-agent httpd
systemctl enable zabbix-server zabbix-agent httpd

#Finish go to http://ip_addr/zabbix
