# Instalación de Apache + Virtual Host

## 1. 

Empezamos creando los ficheros de configuración necesarios para poder arrancar el contenedor de apache. Para ello usaremos los siguientes comandos:

```
$ docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > httpd.conf

```

```
$ docker run --rm httpd:2.4 cat /usr/local/apache2/conf/mime.types > mime.types

```

Esto lo guardamos en el directorio *conf*

Si no otra forma es buscar el script de cada archivo, copiarlo y pegarlo en sus respectivos ficheros que crearemos dentro de la carpeta *conf*.


## 2.

Ahora procedemos a añadir todos los ficheros de configuración para que funcione el servidor DNS. Estos se subirán en el repositorio. . Estarán localizados en el directorio *"configuración"*.

- Empezamos añadiendo en el __docker-compose.yml__ un contenedor **apache**, un **servidor DNS** y un **cliente**. También crearemos la subred para asignárselo a estos contenedores. 

El fichero docker-compose.yml está en el repositorio.

Cuyo script quedará tal que así:

```

services:
  servidor_apache:
    container_name: asir_apache_web
    image: httpd:latest
    ports:
      - "80:80"
    # Mapeamos el directorio raíz (equipo:contenedor)  
    volumes:
      - ./paginas:/usr/local/apache2/htdocs
      - ./conf:/usr/local/apache2/conf   
    networks:
      red:
        ipv4_address: 192.168.1.14

  servidor_dns:
    container_name: asir_servidor_dns
    image: ubuntu/bind9
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      #Mapeo de puertos
    volumes:
      - ./configuracion:/etc/bind
      - ./zonas:/var/lib/bind
      #Para mapear los directorios
    networks:
      red:
        ipv4_address: 192.168.1.1

  
  cliente:
    container_name: asir_cliente_web
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 192.168.1.1
    networks:
      red:
        ipv4_address: 192.168.1.33

networks:
  red:
    external: true

```

Cabe recalcar que no hay que olvidarse de crear un directorio *zonas* donde crearemos los ficheros: "db.fabulasoscuras.int" y "db.fabulasmaravillosas.int". 

Recordar que todos estos archivos están subidos para poder apreciar mejor su contenido.

## 3.

- Creamos otro directorio. En mi caso, lo llamo "paginas" donde crearemos dos ficheros html (También subidos al repositorio).

## 4.

 Finalmente, comprobamos que el contenedor cliente (y desde el) funciona a la perfección con un ***dig***.

 - Para ello haremos los siguientes comandos:

```
dig 192.168.1.1 www.fabulasoscuras.com

dig 192.168.1.1 www.fabulasmaravillosas.com
```

**RECORDATORIO: Para saber si hacemos bien el dig, tenemos que poner la IP del servidor / DNS que se configuró anteriormente en el docker-compose.yml**

## 5. DirectoryIndex , Virtual Host

Empezamos yendo al archivo **httpd.conf** y buscamos un apartado donde pone "Listen 80" en el que debajo pondremos la configuración de los virtual hosts.

Por el lado de la ruta, tienen que ser las carpetas donde están los ficheros .html y aginárselo a los servers que queramos, como puede ser "fabulasoscuras" que está dentro de la carpeta "ww1".

- Ejemplo:
```
Listen 80
<VirtualHost *:80>
    DocumentRoot /usr/local/apache2/htdocs/ww1
    ServerName www.fabulasoscuras.int
    DirectoryIndex prueba.html
</VirtualHost>

```

Una vez hecho y configurado lo anterior, vamos al cliente y descargaremos el Lynx con *apk add lynx*.

Para hacer una consulta con dicho comando, sería --> **lynx www.fabulasoscuras.int:80**