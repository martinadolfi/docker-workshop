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

## Referencias:

https://docs.docker.com/compose/compose-file/

https://docs.docker.com/engine/reference/builder/#usage

https://github.com/docker/labs/tree/master/beginner

https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/

https://hub.docker.com/_/nginx/