# Building images with Dockerfiles

En el ejemplo anterior, se abordo de una forma muy simple empezamos el contenedor interactivamente, corrimos los comandos que queriamos ( como `apt-get install` y `gem install`) y despues hizimos commit del contenedor a una nueva imagen.

En lugar de solo correr comandos y agregar herramientas como `wget`, ahora vamos a poner nuestras instrucciones en un archivo especial llamado `Dockerfile`. Un `Dockerfile` es un concepto similar al de recetas y manifiestos encontrados en otras herramientas de automatización como Chef o Puppet.


El formato se ve así:
```
# Comment
INSTRUCTION arguments
```

Una vez que creemos un `Dockerfile` y agregemos todas las instrucciones, podremos usarlo para crear una imagen usando el comando de `docker build`. El formato de este commando es:

```
docker build [OPTIONS] PATH | URL | -
```

## Ejemplo de crear una imagen a partir de un Dockerfile

```
# Dockerfile
# Super simple example of a Dockerfile
#
FROM ubuntu:latest
MAINTAINER Adrian Rangel "adrian.rangel@gmail.com"

RUN apt-get update
RUN apt-get install -y ruby-full wget
RUN gem install sinatra sinatra-contrib

ADD hello.rb /home/hello.rb

WORKDIR /home
```

Como puedes ver, es bastante sencillo: empezamos de "ubuntu:latest," instalamos las dependencias con el comando de `RUN`, agregamos el archivo con nuestro código usando el comando de `ADD` y establezemos el directorio predeterminado para cuando empieze el contenedor. Una vez que tenemos el `Dockerfile`, podemos crear la imagen usando `docker bild`, así:

```
docker build -t "simple_sinatra:dockerfile" .
```

La opción de `-t` le agrega un tag a la imagen para que tenga un nombre de repositorio y tag. También nota como al final hay un `.`, este le dice a Docker que use el Dockerfile que esta en el directorio actual.

Finalmente, corran el contendor usando el siguiente comando.

```
docker run -p 4567:4567 simple_sinatra:dockerfile ruby hello.rb
```
