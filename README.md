# docker-workshop

## Objetivo

Este workshop tiene como objetivo fijar conceptos de containerización en base a ejemplos prácticos de:

* creación de imágenes.
* instaciación de containers.
* división y separación de servicios.
* creación de arquitecturas compuestas

Para cumplir con este objetivo, utilizaremos las siguientes herramientas:

* docker
* docker registry
* docker-compose
* git

Este workshop no está pensado para hacerlo solo, sino con la guía de un instructor. 

Estas instrucciones por si solas carecen de significado, es solo con la guía de un instructor que se pueden considerar de verdadera utilidad.

## Instalando docker

Aunque este no es el método recomendado para ambientes de producción, si es el más rápido para probar docker, por lo tanto lo utilizaremos:

```bash
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker TU-USUARIO
```

*Se van a  tener que volver a loguear para que apliquen los cambios al grupo.*

Vean las referencias para entender cual es la mejor manera de instalar docker en un ambiente productivo.

## Corriendo mis primeros containers

Corremos un container básico que solo muestra un `hola mundo`

```bash
docker run hello-world
```

Ahora , vamos a hacer algo mas ambicioso, corramos ubuntu.

```bash
docker run -it ubuntu bash
```

Una vez corrido, podremos ver que estaremos como logueados al bash en un ubuntu, sin embargo no habrá nada mas corriendo (intenten un `ps auxf`).

Cuando salimos con `exit` veremos que el container se ha detenido, al igual que el container de hello-world.

Para ver los containers corriendo:

```bash
docker ps
```

Para ver todos los containers, inclusive los que no estan corriendo:

```bash
docker ps -a
```

Para eliminar un container que no queremos utilizar mas:

```bash
docker rm [nombre_del_container|id_del_container]
```

Para eliminar todos los containers:

```bash
docker rm $(docker ps -a -q)
```

## Una cuestión de imagen

Vemos las imágenes que hemos descargado así:

```bash
docker images
```

Eliminamos una imagen con:

```bash
docker rmi [repositorio|id]
```

Eliminamos todas las imágenes con:

```bash
docker rmi $(docker images -q)
```

## Limpieza total

Una ayuda para limpiar sobre todo cuando estamos en ambientes de desarrollo, esto vuela todo lo que no esté en uso:

```bash
docker system prune -f --volumes
```

Y esto vuela TODO:

```bash
docker system prune -a -f --volumes
```

## Creando mi primer imagen

En un directorio separado, creamos un archivo de nombre `Dockerfile` con el siguiente contenido:

```Dockerfile
FROM alpine
CMD echo "hola mundo"
```

Luego, vamos a construir esa imágen:

```bash
docker build -t mi-hola-mundo .
```

Y ahora vamos a crear un container en base a esa imagen:

```bash
docker run --rm mi-hola-mundo
```

*Porque hicimos `--rm`??? porque no nos interesa que persista el container, cuando se termine de ejecutar, se elimina asi no queda dando vueltas (pueden ver que no queda haciendo `docker ps -a`)*

Hagamos una prueba mas, utilicemos el siguiente Dockerfile:

```Dockerfile
FROM alpine
CMD ping 8.8.8.8
```

Luego, vamos a construir esa imágen:

```bash
docker build -t mi-ping .
```

Y ahora vamos a crear un container en base a esa imagen:

```bash
docker run --rm mi-ping
```

Levantemos otra terminal, y veamos con `ps auxf` que es lo que efectivamente está corriendo.

Luego detengamos el container con:

```bash
docker stop [nombre_del_container|id_del_container]
```

Creemos una imagen con mas cosas, supongamos que necesitamos instalar paquetes para que algo funcione, un Dockerfile así no funcionaría:

```Dockerfile
FROM ubuntu
CMD traceroute 8.8.8.8
```

*(si quieren pruebenlo, pero la imagen de ubuntu no trae `traceroute`)*

Por esta razón deberíamos instalarlo de la siguiente manera:

```Dockerfile
FROM ubuntu
RUN apt-get update && apt-get install traceroute -y
CMD traceroute 8.8.8.8
```

Ahora hagamos el build y veamos el proceso:

```bash
docker build -t mi-tracert .
```

Ejecutemos el container y veamos el resultado:

```bash
docker run --rm mi-tracert
```

Intentemos hacer un build nuevamente del Dockerfile, tal y como está repitiendo esto:

```bash
docker build -t mi-tracert .
```

Vemos que todo lo trajo de la cache.

Que diferencia habría si utilizo este `Dockerfile`:

```Dockerfile
FROM ubuntu
RUN apt-get update 
RUN apt-get install traceroute -y
CMD traceroute 8.8.8.8
```

Y si luego cambio solo la linea de instalación?

```Dockerfile
FROM ubuntu
RUN apt-get update 
RUN apt-get install traceroute net-tools -y
CMD traceroute 8.8.8.8
```

Vemos como se crean los layers de las imágenes y como tomar provecho de los mismos.

*TIP: si no queremos que utilice cache para nada, podemos hacer el build con `--no-cache`*

## Creando una imagen un poco mas seria

Imaginemos que queremos tener una web estática, con un nginx sirviendo esa web. 

Tenemos un html con el siguiente contenido que se llama `index.html`:

```html
<html>
<body>
hola, soy una web muy muy estática.
</body>
</html>
```

Creamos un Dockerfile de la siguiente manera:

```Dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
```

*TIP: `COPY` y `ADD` son casi lo mismo, miren la documentación de referencia.*

Lo buildeamos así:

```bash
docker build -t mi-web-estatica .
```

*TIP: el archivo `index.html` y el archivo `Dockerfile` deben estar en el mismo directorio para que esto funcione de esta manera.

Y lo corremos así:

```bash
docker run --rm --name=mi-instancia -d --publish="80:80" mi-web-estatica
```

Podemos ver que está funcionando haciendo:

```bash
curl http://localhost/index.html
```

Podemos ver los logs del container de esta manera:

```bash
docker logs -f mi-instancia
```

Detenemos el container con esto:

```bash
docker stop mi-instancia
```

*TIP: noten que como lo levantamos con `--rm` ademas de detenerlo, lo borra*

*TIP: Pueden utilizar como referencia para crear imagenes complejas las referencias de Dockerfiles de la librería de docker, como por ejemplo el de mariadb, el de wordpress, o cualquier otro, mientras traten de mantener la misma filosofía.*

*TIP (el regreso):tengan en cuenta que no todo el mundo crea los dockerfiles con la filosofía correcta, uno podría crear una imagen de wordpress donde en el mismo container corra el wordpress y la base de datos, y funcionaría, pero no seguiría la filosofía planteada*

## docker-compose

Primero instalaremos docker-compose de la siguiente manera:

```bash
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version

```

Luego creamos un archivo de nombre `docker-compose.yml` con el siguiente contenido.

```yml
version: '3'

services:

  wordpress:
    image: wordpress:4.9.1-php5.6
    restart: unless-stopped
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_PASSWORD: my-db-pass
    volumes:
      - sitedata:/var/www/html

  mysql:
    image: mariadb:10.2.11
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: my-db-pass
    volumes:
      - dbdata:/var/lib/mysql

volumes:
  dbdata:
    driver: local
  sitedata:
    driver: local

```

Y luego para correrlo, lo haremos de la siguiente manera:

```bash
docker-compose up -d
```

Si queremos ver en que estado estan esos containers :

```bash
docker-compose ps
```

*TIP: presten atención a la diferencia con el comando `docker ps`*

Podemos vel el sitio levantado o bien desde un navegador, o bien desde la linea de comandos con:

```bash
curl -I  http://localhost
```

Aqui vemos que está redirigiendo a el script de finalización de configuración de wordpress.

Si queremos ver los logs de los containers:

```bash
docker-compose logs -f
```

Configuremos desde el navegador el sitio, y hagamos un post.

Luego intentaremos, sin perder ningun dato, actualizar la imagen de wordpress que está corriendo, para eso, editaremos el `docker-compose.yml` que habíamos creado y cambiaremos la imagen de esta:

```yml
    image: wordpress:4.9.1-php5.6
```

a esta:

```yml
    image: wordpress:4.9.1-php7.2
```

Para actualizar el container que está corriendo, hacemos:

```bash
docker-compose up -d
```

Veamos nuevamente el sitio y veremos que los datos se han mantenido, e inclusive ni siquiera perdimos la sesión.

Con esto podemos ver los volumenes creados:

```bash
docker volume ls
```

Ahora detengamoslo:

```bash
docker-compose stop
```

*TIP: con `start` vuelve a levantar*

Y ahora, matemoslo definitivamente:

```bash
docker-compose down
```

## Overrides

Probemos para el caso del wordpress, un ejemplo donde en producción hay que cambiarle las claves de conexión a la base de datos. Para eso haremos un nuevo archivo `docker-compose-prod.yml` con el siguiente contenido:

```yml
version: "3"
services:
 wordpress:
    environment:
      WORDPRESS_DB_PASSWORD: my-db-pass2
 mysql:
    environment:
      MYSQL_ROOT_PASSWORD: my-db-pass2
```

Por las dudas, quitaremos todos los volumenes existentes:

```bash
docker volume prune
```

Ahora levantaremos los servicios, pero utilizando la combinación de ambos  docker-compose.yml

```bash
docker-compose -f docker-compose.yml -f docker-compose-prod.yml up -d
```

*TIP: debemos tener en cuenta que en estos casos, cada vez que usamos `docker-compose` debemos hacerlo con todos los `-f`, por ejemplo para `logs`, para `ps`, etc.*

Si utilizamos docker inspect para ver como quedaron las variables de entorno, las encontraremos así:

```bash
docker inspect workshop_wordpress_1 | grep PASSWORD
docker inspect workshop_mysql_1 | grep PASSWORD
```

*TIP: en el comando de acá arriba, la palabra `workshop` es por el directorio en donde está parado*

## docker stack (swarm y otras magias)

Vamos a crear una aplicación donde podamos manejar varias replicas de una imagen

Iniciemos un swarm con:

```bash
docker swarm init
```

Creamos un archivo `docker-compose.yml` de con el siguiente contenido:

```yml
version: "3"
services:
  web:
    image: mi-web-estatica
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:

```

Levantamos el servicio con:

```bash
docker stack deploy -c docker-compose.yml mystaticsite
```

Verificamos lo que esta levantado con:

```bash
docker service ls
```

Vemos el log de la aplicación con:
```bash
docker service logs -f mystaticsite_web
```

Mirando los logs, naveguemos para corroborar que efectivamente le pega a diferentes containers.

Ahora quitaremos este stack con:

```bash
docker stack rm mystaticpage
```

Si queremos salir del swarm hacemos:

```bash
docker swarm leave --force
```

Como desafío adicional, probamos repetir el procedimiento con wordpress.

```yml
version: "3"
services:
 wordpress:
    image: wordpress:4.9.1-php5.6
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_PASSWORD: my-db-pass
    volumes:
      - sitedata:/var/www/html
    networks:
      - webnet
      - backend
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
 mysql:
    image: mariadb:10.2.11
    environment:
      MYSQL_ROOT_PASSWORD: my-db-pass
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - backend
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

networks:
  webnet:
  backend:

volumes:
  dbdata:
    driver: local
  sitedata:
    driver: local

```



## Referencias:

https://docs.docker.com/compose/compose-file/

https://docs.docker.com/engine/reference/builder/#usage

https://docs.docker.com/get-started/

https://github.com/docker/labs/tree/master/beginner

https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/

https://hub.docker.com/_/nginx/

https://docs.docker.com/compose/install/

https://hub.docker.com/_/wordpress/

https://hub.docker.com/_/mariadb/

