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
##### 
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