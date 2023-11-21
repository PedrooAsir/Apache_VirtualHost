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

Si no otra forma es buscar el script de cada archivo, copiarlo y pegarlo en la carpeta creada "conf" donde dentro de ella se crean estos dos ficheros.


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

- Finalmente, comprobamos que el contenedor cliente funciona a la perfección con un ***dig***.

 - Para ello haremos los siguientes comandos:

```
dig 192.168.1.1 www.fabulasoscuras.com

dig 192.168.1.1 www.fabulasmaravillosas.com


```