#PRACTICA 1 - DNS Linux
###Nerea Pascual Garcia

Creamos una carpeta llamada **Practica1** con 2 subcarpetas en su interior: **config** y **zonas**.

Creamos el fichero **docker-compose.yml** en DNS:
~~~
version: "3.9"  # optional since v1.27.0
services:
  bind9:
    container_name: practica1_bind9
    image: internetsystemsconsortium/bind9:9.16
    ports:
      - 5300:53/udp
      - 5300:53/tcp
    volumes:
      - /home/asir2a/Escritorio/SRI/DNS/config:/etc/bind
      - /home/asir2a/Escritorio/SRI/DNS/zonas:/var/lib/bind
~~~

Usamos el comando: *docker-compose up*.

Creamos un fichero llamado **named.config** dentro de la carpeta config con el siguiente contenido:
~~~
options {
        directory "/var/cache/bind";
        listen-on { 127.0.0.1; };
        listen-on-v6 { ::1; };
        allow-recursion {
                none;
        };
        allow-transfer {
                none;
        };
        allow-update {
                none;
        };
};

zone "asircastelaop1.com." {
        type primary;
        file "/var/lib/bind/db.asircastelaop1.com";
        notify explicit;
};
~~~

Usamos el comando: *docker-compose start*. El contenedor se quedará arrancado.

Para la configuración de la zona nos iremos a la carpeta llamada zonas y crearemos un fichero llamado **db.asircastelaop1.com** donde incluiremos los siguiente:
~~~
$TTL    3600
@       IN      SOA     ns.asircastelaop1.com. nerea.danielcastelao.org. (
                   2007010401           ; Serial
                         3600           ; Refresh [1h]
                          600           ; Retry   [10m]
                        86400           ; Expire  [1d]
                          600 )         ; Negative Cache TTL [1h]
;
@       IN      NS      ns.asircastelaop1.com.
@       IN      MX      10 servidorcorreo.asircastelaop1.org.

ns     IN      A       34.12.1.1
etch    IN      A       123.123.4.5
pop     IN      CNAME   ns
www     IN      CNAME   etch
mail    IN      CNAME   etch
~~~

Usamos el comando: *docker-compose start*. 
Para comprobar que funciona correctamente nos iremos a los logs, donde debería haberse cargado la zona correctamente.

Modificamos el fichero docker-compose.yml y añadiremos lo siguiente a la altura de volumes:
~~~
networks:
   bind9_subnet:
      ipv4_address: 10.1.0.254
 ~~~     
y a la altura de services:
~~~
networks:
  bind9_subnet:
    external: true
~~~

Eliminamos el contenedor con el comando: *docker-compose down -v* y lo volvemos a arrancar con: *docker-compose up*.

Entramos en el fichero named.conf y modificamos su contenido por el siguiente:
~~~
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
~~~

En la carpeta config creamos 2 ficheros llamados **named.conf.options** y **named.conf.local**.

En el fichero named.conf.options incluiremos lo siguiente:
~~~
options {
    directory "/var/cache/bind";

    forwarders {
        8.8.8.8;
        8.8.4.4;
    };
    forward only;

    listen-on { any; };
    listen-on-v6 { any; };
    allow-query {
        any;
    };
};
~~~

En el fichero named.conf.local incluiremos lo siguiente:
~~~
zone "asircastelaop1.com." {
        type primary;
        file "/var/lib/bind/db.asircastelaop1.com";
        notify explicit;
};
~~~

Usamos el comando: *docker-compose start* y el contenedor debería quedarse encendido.

En el fichero db.asircastelaonerea.com modificaremos lo siguiente:
En la linea de ns modificar por: ns IN A 10.1.0.254 (la IP que pusimos en networks del docker-compose.yml)

Además añadir:
~~~
test    IN      A       10.1.0.2
alias   IN      CNAME   test
~~~

Modificaremos el docker-compose.yml y añadiremos lo siguiente a la altura de bind9:
~~~
asir_cliente:
      container_name: asir_cliente
      image: alpine
      networks:
        - bind9_subnet
      stdin_open: true
      tty: true
      dns:
        - 10.1.0.254
~~~

El fichero quedará de la siguiente forma:
~~~
version: "3.9"  # optional since v1.27.0
services:
  bind9:
    container_name: practica1_bind9
    image: internetsystemsconsortium/bind9:9.16
    ports:
      - 5300:53/udp
      - 5300:53/tcp
    volumes:
      - /home/asir2a/Escritorio/SRI/DNS/config:/etc/bind
      - /home/asir2a/Escritorio/SRI/DNS/zonas:/var/lib/bind
    networks:
      bind9_subnet:
        ipv4_address: 10.1.0.254
  asir_cliente:
    container_name: asir_cliente
    image: alpine
    networks:
      - bind9_subnet
    stdin_open: true
    tty: true
    dns:
      - 10.1.0.254
networks:
  bind9_subnet:
    external: true
~~~

Usamos *docker-compose down -v* y *docker-compose up* de modo que se crea el contenedor alpine y se quedará levantado.

En el apartado de contenedores nos vamos a donde se encuentra alpine, seleccionamos la opción inspect y copiamos la id.
Usamos el comando *docker exec -it idAlpine sh*.
Por último, hacemos ping ns.asircastelaop1.com y debería hacer ping.