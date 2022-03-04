Apuntes Examen DAW

## Eduardo Martínez Romero
### 2ºDAW 2021/2022 - IES Doñana
***
# 00-installer-confg.yaml <small>(Netplan)</small>
```
network:
	ethernets:
		enp0s3:
			dhcpd4: true
		enp0s8:
			dhcp4: false
			dhcp6: false
			addresses: [192.168.168.1/24]
			nameservers:
				addresses: [1.1.1.1, 1.0.0.1]
		version: 2
```

# dhcpd.conf
```
[...]
# Subnet privada 1
subnet 192.168.168.0 netmask 255.255.255.0 {
	range 192.168.168.100 192.168.168.150;
	option domain-name-servers 192.168.168.1;
	option domain-name "aulap7.tk";
	option subnet-mask 255.255.255.0;
	option-boradcast-address 192.168.168.255;
	default-lease-time 300;
	max-lease-time 3600;
}

# IP reservada
host cliente1 {
	hardware ethernet 08:00:07:26:c0:a5;
	fixed-address 192.168.168.200;
}
```

# named.conf.local
```
[...]
// Zona aulap7.tk
zone aulap7.tk {
	type master;
	notify no;
	allow-transfer {none;};
	file "/etc/bind/db.aulap7.tk";
};
```

# db.aulap7.tk
```
; BIND data file for aulap7.tk
$TTL	604800
$ORIGIN aulap7.tk.
@	IN	SOA	servidor1.aulap7.tk.	admin.aulap7.tk. (
						2022022401
							604800
							 86400
						   2419200
							604800 )
;
@		IN	NS	servidor1.aulap7.tk.
;
servidor1	IN	A	192.168.168.1
servidor2	IN	A	192.168.168.2
;
www		IN	CNAME	servidor1
web		IN	CNAME	servidor1
```

# aulap7.conf
```
<VirtualHost *:80>
	ServerName www.aulap7.tk
	ServerAlias web.aulap7.tk
	ServerAdmin emartinez@gmail.com
	DocumentRoot /var/www/aulap7
	
	LogLevel debug
	ErrorLog ${APACHE_LOG_DIR}/aulap7.error.log
	CustomLog ${APACHE_LOG_DIR}/aulap7.access.log combined
	
	ErrorDocument 401 /errores/401.html
	ErrorDocument 403 /errores/403.html
	ErrorDocument 404 /errores/404.html
	Redirect "/moreinfo" "https://www.iesdonana.org/"
	
	<Directory /var/www/aulap7>
		Require all granted
		DirectoryIndex main.htm main.html
		Options +Indexes -FollowSymLinks
	</Directory>
	
	<Directory /var/www/aulap7/privada1>
		AuthType Digest
		AuthName "directivos"
		AuthUserFile "/etc/apache2/.aulap7_digest"
		Require user ana carlos
	</Directory>
	
	<Directory /var/www/aulap7/privada2>
		AuthType Digest
		AuthName "apoderados"
		AuthUserFile "/etc/apache2/.aulap7_digest"
		<RequireAll>
			Require all granted
			Require user roberto isabel
			Require not ip 192.168.168.0/24
		</RequireAll>
	</Directory>
	
	Alias /web /home/emartinez/web
	<Directory /home/emartinez/web>
		Require all granted
		AllowOverride All
	</Directory>
</VirtualHost>
```

# .htaccess
```
DirectoryIndex index.html
Options -Indexes
```

# Comandos útiles
## Sistema
- Control de servicios: `systemctl [OPCION] [SERVICIO]`
- Consultar nombre del host: `hostname`
- Cambiar nombre del host: `hostnamectl set-hostname [NOMBRE]`
- Mostrar árbol de directorios: `tree`
- Crear varios directorios a la vez: `mkdir -p asignaturas/{mates,lengua}`
- Abrir varios ficheros de texto a la vez: `nano [FICHERO] [FICHERO]`
- Crear enlaces simbólicos: `ln -s [OBJETIVO] [NOMBRE]`
- Manual de un comando: `man [COMANDO]`

## Red
- Mostrar info de un adaptador: `ip -c addr show [ADAPTADOR]`
- Activar/Desactivar un adaptador: `ip link set [ADAPTADOR] [up/down]`
- Renovar direcciones DHCP: `dhclient -v`
- LIberar direcciones DHCP: `dhclient -v -r`
- Listar DNS del sistema: `systemd-resolve --status`
- Listar DNS específica: `systemd-resolve [DOMINIO]`
- Limpiar caché DNS: `systemd-resolve --flush-caches`

## Apache
- Activar/Desactivar sitio: `[a2ensite/a2dissite] [SITIO]`
- Activar/Desactivar módulo: `[a2enmod/a2dismod] [MODULO]`
- Comprobar si un sitio está activo: `ls [SITES_EN_DIR] | grep [SITIO]`
- Comprobar si un mod está activo: `ls [MODS_EN_DIR] | grep [MODULO]`
- Crear usuario Basic: `htpasswd [-c*] [RUTA] [USUARIO]`
- Crear usuario Digest: `htdigest [-c*] [RUTA] [REALM] [USUARIO]`
