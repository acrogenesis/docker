# Sinatra Docker demo

App básica de Sinatra. Haremos:

- Iniciar un contenedor llamado "simple_sinatra", basado en Ubuntu
- Instalar dependencias
- Instalar la aplicación en sí (que es sólo un "Hello World!")
- Commit el contenedor a una nueva imagen para nuestra aplicación
- Iniciar un nuevo contenedor basado en la imagen
- Accesar la aplicación desde el navegador

Nuestro primer paso es hacer un nuevo contenedor basado en `ubuntu:latest`. Lo haremos en modo interactivo usando las opciones `-it`, y le daremos un nombre:

```
docker run -it --name="simple_sinatra" ubuntu:latest /bin/bash
```
## Instalación de dependencias

Ahora que tenemos el contenedor corriendo, instalaremos las dependencias que necesitamos. Ya que Sinatra es un micro framework de Ruby, vamos a ver qué versión de Ruby tenemos:

```
root@c02b97c4e0c8:/# ruby -v
bash: ruby: command not found
```
Ruby normalmente viene instalado en Ubuntu. Pero, para ahorrar espacio, las imagenes base de Docker son versiones más ligeras que las oficiales.

Entonces vamos a instalar el ambiente de ruby.

```
apt-get update
apt-get install -y ruby-full wget
```

Ahora instalemos sinatra

```
gem install sinatra sinatra-contrib
```

Ya instalamos muchas cosas, hagamos un commit de la imagen con un simple mensaje:

```
docker commit -m "installed ruby and sinatra" simple_sinatra
2de1b57eb6eb...
```

Ahora si checkas `docker images` veras un nuevo repositorio:

```
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              2de1b57eb6eb        2 days ago          355.4 MB
ubuntu              latest              d2a0ecffe6fa        2 weeks ago         188.4 MB
```

Pero, no tiene nombre -- solo vemos `<none>`. Podemos usar un `docker tag` para darle un nombre a nuestra imagen, asi:

```
docker tag 2de1b57eb6eb simple_sinatra
```
```
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
simple_sinatra      latest              2de1b57eb6eb        3 minutes ago       355.4 MB
ubuntu              latest              d2a0ecffe6fa        2 weeks ago         188.4 MB
```

Finalmente, hechemos un vistazo al historial que nos da.  Hemos cargado alredodor de 168MB de cosas nuevas a nuestra imagen:

```
docker history simple_sinatra
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
2de1b57eb6eb        5 minutes ago       /bin/bash                                       167 MB              installed ruby and sinatra
d2a0ecffe6fa        2 weeks ago         /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B
29460ac93442        2 weeks ago         /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.895 kB
b670fb0c7ecd        2 weeks ago         /bin/sh -c echo '#!/bin/sh' > /usr/sbin/polic   194.5 kB
83e4dde6b9cf        2 weeks ago         /bin/sh -c #(nop) ADD file:c8f078961a543cdefa   188.2 MB
```

### Instalación del código
Primero, arranquemos un nuevo contenedor:

```
docker run -it -p 4567:4567 simple_sinatra /bin/bash
```
Como antes, usaremos la opción `-it` para hacerlo interactivo, pero ahora estamos agregando un par de opciones nuevas:

La opción `-p 4567:4567` expone el puerto 4567 en el contenedir y rutea al puerto 4567 del host. Si estas corriendo Docker en una Mac, hay que exponer el puerto 4567 del VM al host "real". Porque docker esta corriendo en la maquina virtual entonces esta es su host. Tenemos que exponer el puerto 4567 de la VM a la mac.

Ahora bajemos un simple hello world de sinatra del repo de este curso.

```
cd home/
wget https://raw.githubusercontent.com/acrogenesis/docker-hs/master/01/hello.rb
...
```
```
root@daaf59da11c2:/home# cat hello.rb
require 'sinatra'
require 'sinatra/reloader'

set :bind, '0.0.0.0'

get '/' do
  'Hello world!'
end
```
Finalmente, iniciemos la app. Veremos algo asi:
```
root@daaf59da11c2:/home# ruby hello.rb
[2015-07-20 03:45:11] INFO  WEBrick 1.3.1
[2015-07-20 03:45:11] INFO  ruby 1.9.3 (2013-11-22) [x86_64-linux]
== Sinatra (v1.4.6) has taken the stage on 4567 for development with backup from WEBrick
[2015-07-20 03:45:11] INFO  WEBrick::HTTPServer#start: pid=16 port=4567
```

### Viendo la app de sinatra en nuestra máquina
Veamos si funciona... En la terminal del host corre el siguiente comando:
```
curl 127.0.0.1:4567
```

Si estas corriendo estos en una Linux como host, deberias de ver un `Hello World!`. Si lo estas corriendo en boot2docker en una Mac o Windows, muy probablemente te dio este error:
```
curl: (7) Failed to connect to 127.0.0.1 port 4567: Connection refused
```
Cuando empezaste este contenedor y le dijiste que expusiera el puerto 4567, expusiste el de la maquina virtual, no el del host. Necesitamos usar VBoxManage para abrir un puerto en el VM al host, asi:

```
VBoxManage controlvm boot2docker-vm natpf1 "sinatra-server,tcp,127.0.0.1,4567,,4567"
```
Ahora debes de poder conectarte a nuestra simple app así:
```
curl 127.0.0.1:4567
Hello world!
```
