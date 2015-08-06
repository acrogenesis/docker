DokerHub es un sitio donde puedes guardar y compartir las imagenes que creeas. Esta modelado como GitHub. Ofrece cosas como:

- Hosting de imagenes de Docker. Como en Github hay imagenes publicas que son gratis y si quieres imagenes privadas cuestan.
- Colaboradores que pueden dar push o pull a las imagenes y Webhooks para cada vez que subas una imagen.
- Estadisticas sobre la imagen, como descargas y estrellas.

Hay dos formas de interactuar con Docker Hub:

- La interfaz web, donde puedes registrar metadatos de la imagen (descripción, etc) y agregar quitar coloaboradores, etcetera. Es muy similar a Github.
- La linea de comandos de docker, donde puedes dar pull, push o buscar imagenes. Es similar a git.

## Subir nuestra imagen de sinatra a Docker

Primer hay que crear una cuenta en Docker Hub, se puede crear desde el sitio web o desde la terminal. Hagan signup en [Docker Hub](https://hub.docker.com/account/signup/)

Una vez que hayan hecho su cuenta corran. (Támbien pueden registrarse con este comando si no quieren visitar el sitio web)
```
docker login
```

Intentemos subir nuestra imagen
```
docker push simple_sinatra:latest

You cannot push a "root" repository. Please rename your repository to <user>/<repo> (ex: acrogenesis/simple_sinatra)
```

Este error sucede porque no tenemos permisos para subir imagenes a `root`.

En Docker hay dos niveles de namespaces:
- Root: Este namespace esta reservado para imagenes `root` que estan mantenidas directamente por Docker Inc. Basicamente es un espacio vacio para imagenes como `ubuntu` o `debian`.
- User: Este namespace es para los usuarios que usan Docker Hub, asi como `acrogenesis/simple_sinatra` o `crashsystems/gitlab-docker`

Entonces tenemos que agregarle nuestro username a la imagen usando el comando de `docker tag`

```
docker tag simple_sinatra acrogenesis/simple_sinatra
```

Una vez que hallamos retagged nuestra imagen ya la podemos subir a Docker Hub

```
docker push acrogenesis/simple_sinatra:latest
```

Listo esta imagen ya la puedes compartir con tus seres queridos o compañeros de trabajo.
Esta en Docker Hub en un URL como https://hub.docker.com/u/acrogenesis/simple_sinatra/

# Descargar una imagen para uso con Rails
Vamos a buscar una imagen para una de nuestras apps de rails.

```
docker search rails
```
```
NAME                          DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
rails                         Rails is an open-source web application fr...   240       [OK]
finnlabs/rails                Base image for Rails applications followin...   5                    [OK]
spartan/rails                                                                 2                    [OK]
erikap/passenger-rails        Hosting a Rails production app in Phusion ...   2                    [OK]
lucio/rails                   Latest Ruby on Rails on Ubuntu Trusty           2                    [OK]
shicholas/rails                                                               1                    [OK]
tenstartups/rails                                                             1                    [OK]
voxxit/rails                                                                  1                    [OK]
foodcare/rails                                                                1                    [OK]
dharmamike/pmx-rails          This is a base Rails app meant only for ex...   1                    [OK]
cloudgear/rails               Smallest Rails optimized image with build ...   0                    [OK]
feduxorg/centos-rails                                                         0                    [OK]
bantl23/rails                 rails development                               0                    [OK]
quirky/rails                                                                  0                    [OK]
shakr/rails                   Rails base image for docker                     0                    [OK]
minimum2scp/rails4            Rails 4.x and RDBMS client, headers instal...   0                    [OK]
centurylink/alpine-rails      Lightweight Rails image based on Alpine Linux   0                    [OK]
zlanger/jenkins-rails-slave   ubuntu based jenkins slave for running rai...   0                    [OK]
zazo/rails                                                                    0                    [OK]
carsocial/rails                                                               0                    [OK]
coalwater/docker-rails        A rails development environment                 0                    [OK]
jmorales/rails                Rails onbuild fork supporting entrypoints       0                    [OK]
bluk/docker-rails                                                             0                    [OK]
zumbrunnen/rails              Ruby on Rails image, using Passenger Stand...   0                    [OK]
ontouchstart/rails                                                            0                    [OK]
```

Esto nos regresa las siguientes columnas
- Name: Es el nombre y el namespace de la imagen
- Description: Los metadatos que tiene esta imagen en Docker Hub
- Stars: El numero de personas que le han dado "star" a esta imagen.
- Official: Si viene o no de Docker Inc. Un `OK` se despliega en esta columna, nos ayuda a darnos una sensación de seguridad de que esta imagen a examinada.
- Automated: Aunque menos autoritativo que una imagen oficial las imagenes que tengan esto signfica que la imagen se constryo automatizadamente y provee un link a Github o Bitbucket donde puedes revisar el contenido de la imagen antes de usarla en tu ambiente.

### Descargando la imagen

```
docker pull rails
```

Ahora ya tenemos una imagen lista para correr rails.

Vamos a usar una app de ejemplo para probar esta app

```
git clone git@github.com:acrogenesis/learn-rails.git
cd learn-rails
```

Creemos un Dockerfile y agregemosle la fuente de la imagen

```
FROM rails:onbuild
```

Esta imagen automaticamente va a copiar los contenidos de `.` a `/usr/src/app`, correra `bundle install`, expondra el puerto `3000` y correra automaticamente `rails server`.

Hagamos el build de nuestra imagen
```
docker build -t learn-rails .
```

Ahora a correr nuestro conetnedor
```
docker run --name learn-rails -p 3000:3000 -d learn-rails
```

Ahora ingresemos a `localhost:3000` y podremos ver que nuestra app ya esta corriendo. (Recuerden los que esten usando Mac de abrir el puerto de VirtualBox a la Mac)
