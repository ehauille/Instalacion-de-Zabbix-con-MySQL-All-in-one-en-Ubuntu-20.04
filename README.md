# Instalación-de-Zabbix-con-MySQL-All-in-one-en-Ubuntu-20.04
 
# Ambiente utilizado
	- Servidor Zabbix: MySQL / Zabbix Server / Frontend
	- Sistema Operacional: Ubuntu 20.04
	- Memória: 4GB de RAM
	- CPU: 2 vCPU
	- Disco: 50GB
 
# Verificar timezone actual
	timedatectl status
 
# Definir timezone
	dpkg-reconfigure tzdata (seleccionar tu timezone)
 
# Verificar alteración
	timedatectl status
	
# Verificar día y hora
	date
    
# Instalar utilitários
	apt install -y net-tools
 
# Instalando Mysql 8
# Verificar versión disponible en el repositorio
	apt search mysql-server
 
Output
 
Sorting... Done
Full Text Search... Done
mysql-server/bionic-security,bionic-updates 5.7.30-0ubuntu0.18.04.1 all
MySQL database server (metapackage depending on the latest version)
mysql-server-5.7/bionic-security,bionic-updates 5.7.30-0ubuntu0.18.04.1
amd64
MySQL database server binaries and system database setup
mysql-server-core-5.7/bionic-security,bionic-updates 5.7.30-
0ubuntu0.18.04.1 amd64
MySQL database server binaries
 
# Instalar el paquete de mysql server
	apt install mysql-server mysql-client -y
 
# Habilitar e iniciar servicio de MySQL
	systemctl enable --now mysql
	systemctl status mysql
 
# Definir usuario y contraseña root de MySQL
	mysql_secure_installation
 

Securing the MySQL server deployment.
Connecting to MySQL using a blank password.
VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?
- Press y|Y for Yes, any other key for No: y


There are three levels of password validation policy:
LOW Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary
file
Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG:
Please set the password for root here.


New password: password
Re-enter new password: password
Estimated strength of the password: 100
 
- Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y


By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

- Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
- Success.
 
Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.
 
- Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
- Success.


By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.
 
- Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 
- Dropping test database...
- Success.
- Removing privileges on test database...
Success.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.
All done!
'''
 
# Conectar, crear la base de datos y el usuário de Zabbix
	mysql -u root -p
	create database zabbix character set utf8 collate utf8_bin;
	create user 'zabbix'@'localhost' identified by 'Password';
	grant all privileges on zabbix.* to 'zabbix'@'localhost';
	flush privileges;
	exit;
 
	character set utf8 - suporte a multilinguagem collate utf8_bin - armazena os
 
# Instalar el repositorio oficial de Zabbix Server
 
	wget https://repo.zabbix.com/zabbix/5.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.0-1+focal_all.deb
	dpkg -i zabbix-release_5.0-1+focal_all.deb
	apt update
 
# Instalando Zabbix Server
	apt install zabbix-server-mysql
 
# Cargar esquema inicial de la base de datos
	zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -u (password) -p zabbix
	
# Verificar si la base de datos fue creada
	mysql -u zabbix -p zabbix
	use zabbix;
	show tables;
	quit;
 
Output:
...
166 rows in set (0.01 sec)
 
# Ingresar en el siguiente camino para agregar la contraseña en la base de datos
	vim /etc/zabbix/zabbix_server.conf
 
	 DBPassword
	 Database password.
	 Comment this line if no password is used.

	 Mandatory: no
	 Default:
	 DBPassword=Password
 
# Habilitar servicio e iniciarlo
	systemctl enable --now zabbix-server
	systemctl status zabbix-server
 
# Verifique los logs de Zabbix Server y verifique si tiene errores
	tail -n50 /var/log/zabbix/zabbix_server.log
 
# Instalar frontend y los paquetes
	apt install zabbix-frontend-php zabbix-apache-conf
 
# Configurando PHP
	vim /etc/zabbix/apache.conf
	php_value[date.timezone] = 'timezone'
 
# Habilitar servicio e iniciarlo
	systemctl enable --now apache2
	systemctl restart apache2
 
# Crear regla en el firewall
	ufw allow 80/tcp
	ufw reload
 
# Acesse a la interfaz web
	http://IP_OU_DNS/zabbix
	Usuário: Admin
	Contraseña: zabbix
