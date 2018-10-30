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
<dl>
	<dd>SELINUX=disabled</dd>
</dl>

`setenforce 0`<br/>
`reboot`<br/>

#### apache <br/>
`yum install -y httpd` <br/>
`systemctl enable httpd` <br/>
`systemctl start httpd` <br/>
`netstat -tulnp | grep httpd` <br/>

#### php71 <br/>
`yum install epel-release` <br/>
`rpm -Uhv http://rpms.remirepo.net/enterprise/remi-release-7.rpm` <br/>
`yum install yum-utils` <br/>
`yum-config-manager --enable remi-php71` <br/>
`yum install php71 php-fpm php-cli php-mysql php-gd php-ldap php-odbc php-pdo php-pecl-memcache php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap php-bcmath` <br/>
`systemctl start php-fpm` <br/>
`systemctl enable php-fpm` <br/>
`netstat -tulpn | grep php-fpm` <br/>

#### postgresql11 <br/>
`yum install https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/pgdg-centos11-11-2.noarch.rpm` <br/>
`yum install postgresql11 postgresql11-server` <br/>
`/usr/pgsql-11/bin/postgresql-11-setup initdb` <br/>
`systemctl enable postgresql-11` <br/>
`systemctl start postgresql-11` <br/>
`nano /var/lib/pgsql/11/data/pg_hba.conf` <br/>
<dl>
	# "local" is for Unix domain socket connections only <br/>
	local   all             all                                     trust <br/>
	# IPv4 local connections: <br/>
	host    all             all             127.0.0.1/32            trust <br/>
	# IPv6 local connections: <br/>
	host    all             all             ::1/128                 trust <br/>
</dl>

`systemctl reload postgresql-11` <br/>

#### create user and base zabbix postgresql<br/>
`su - postgres`<br/>
`createuser zabbix -P`<br/>
`createdb -O zabbix zabbix`<br/>
`exit`<br/>

#### install zabbix<br/>
`rpm -i https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm`<br/>
`yum install zabbix-server-pgsql zabbix-web-pgsql zabbix-agent`<br/>
`zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | sudo -u zabbix psql zabbix`<br/>
`nano /etc/zabbix/zabbix_server.conf`<br/>
	DBHost=localhost<br/>
	DBName=zabbix<br/>
	DBUser=zabbix<br/>
	DBPassword=zabbix<br/>
`nano /etc/httpd/conf.d/zabbix.conf`<br/>
	#change (<)IfModule mod_php5.c(>) to (<)IfModule mod_php7.c(>)<br/>
	php_value date.timezone Asia/Almaty<br/>
`systemctl restart zabbix-server zabbix-agent httpd`<br/>
`systemctl enable zabbix-server zabbix-agent httpd`<br/>

#### Finish go to http://ip_addr/zabbix<br/>
