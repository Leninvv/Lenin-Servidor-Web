# Práctica servidor WEB

#### Descripción Docker-compose

##### Creamos el contenedor asir_apache
##### Usamos la imagen httpd:latest
##### La red usada será apanet con una ip estática 192.168.22.4
##### Mapeo el puerto 8081 de la máquina host al puerto 80 del contenedor
##### Ruta del volumen apache-data:/usr/local/apache2

```

version: "3.3"
services:
  asir_apache:
    image: httpd:latest
    networks:
       apanet:
        ipv4_address: 192.168.22.4
    ports:
      - 8081:80
    volumes:
      - apache-data:/usr/local/apache2/htdocs
      - apache-conf:/usr/local/apache2/conf
```
#### Contenedor DNS
##### Creación de contenedor asir_bind9
##### Usamos la imagen internetsystemsconsortium/bind9:9.16
##### La red usada será apanet con una ip estática 192.168.22.5
##### Mapeo el puerto 53 del host al 53 del contenedor
##### Volúmenes a utilizar: conf:/etc/bind options:/var/cache/bind secondaryzones:/var/lib/bind logfiles:/var/log el volumen importante es conf
```

  asir_bind9:
    image: internetsystemsconsortium/bind9:9.16
    networks:
       apanet:
        ipv4_address: 192.168.22.5
    ports:
      - 53:53
    volumes:
      - conf:/etc/bind
      - options:/var/cache/bind
      - secondaryzones:/var/lib/bind
      - logfiles:/var/log
```
#### Contenedor cliente
##### Creación contenedor asir_cliente
##### Usamos la imagen kasmweb/desktop:1.10.0-rolling
##### La red usada será apanet con el dns apuntando a la ip 192.168.22.5
##### Mapeo el puerto 6901 del host y del contenedor porque así lo indica la documentación de la imagen.
##### Parámetro environment para indicar la contraseña para conectarse al cliente
```
  asir_cliente:
    image: kasmweb/desktop:1.10.0-rolling
    networks:
      - apanet
    dns:
      - 192.168.22.5
    ports:
      - 6901:6901
    environment:
      - VNC_PW=123456
    stdin_open: true  # docker run -i
    tty: true         # docker run -t
```
#### Contenedor wireshark
##### Usamos la imagen lscr.io/linuxserver/wireshark
##### Configuramos los siguientes campos como lo especifica la documentación de la imagen
```
  asir_wireshark:
    image: lscr.io/linuxserver/wireshark
    cap_add:
      - NET_ADMIN
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - wiresharkconf:/config
    ports:
      - 3000:3000
    restart: unless-stopped
```
#### Red y volúmenes
##### Indicamos que la red apanet es una red externa que ya existía previamente
##### Indicamos los volúmenes, a su vez indicamos que los volúmenes apache-data, apache-conf y conf ya han sido creados previamente
```

networks:
  apanet:
    external: true

volumes:
  apache-data:
    external: true
  conf:
    external: true
  apache-conf:
    external: true
  options:
  secondaryzones:
  logfiles:
  wiresharkconf:

  ```
  #### Configuración Virtual hosts y DNS
  ##### Habilitamos los Virtual Hosts en httpd.conf

```
# Virtual hosts
Include conf/extra/httpd-vhosts.conf #hay que descomentar esta línea

```

##### Creamos los Virtual Hosts en Httpd-vhosts.conf

```
<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/usr/local/apache2/htdocs/other"
    ServerName apache.com
    ServerAlias other.apache.com
    ErrorLog "logs/dummy-host.example.com-error_log"
    CustomLog "logs/dummy-host.example.com-access_log" common
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/usr/local/apache2/htdocs/otro"
    ServerName apache.com
    ServerAlias otro.apache.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>

```
##### Configuramos los Virtual Hosts en el Bind dns
##### Creamos la zona, los forwardes y el CNAME
##### BIND file de apache.com

```
;
; BIND data file for apache.com
;
$TTL	604800
@	IN	SOA	apache.com. root.apache.com. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	ns.apache.com.
@	IN	A	192.168.22.4
@	IN	AAAA	::1
ns  IN  A	192.168.22.4
@	IN	TXT	"Texto de ejemplo"
www CNAME apache.com. 
other CNAME apache.com.
otro CNAME apache.com.

```

##### Zona en namec.conf.local

```
zone "apache.com" {
    type master;
    file "/etc/bind/db.apache.com";
};

```
##### Configuramos los forwarders en named.conf.options

```
forwarders {
		8.8.8.8;
		8.8.4.4;
	 };

```


