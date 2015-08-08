# Deploy rails con Docker

### docker-compose
Esta herramienta hará que sea muy fácil de correr varios contenedores Docker que funcionan todos juntos. Nos permitirá correr Rails y postgres (cualquier bd) con un comando.

```
curl -L https://github.com/docker/compose/releases/download/1.3.3/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

Creemos una carpeta nueva para la app de rails y un archivo de docker
```
mkdir rails-dc && cd rails-dc
vim Dockerfile
```

Dockerfile
```
FROM ruby:2.2-onbuild
```

Ahora creemos un `Gemfile` con la siguiente inforomación:
```
source 'https://rubygems.org'

gem 'rails'
```

Este ejemplo lo haremos como si nuestra computadora no tuviera ruby instalado o no tuviera la misma versión que el proyecto requiere.

Entonces como generamos el `Gemfile.lock`?

### Lo generamos desde Docker!

```
docker run -it -v $(pwd):/usr/src/app -w /usr/src/app ruby:2.2 bundle install
```

Ya debemos tener los siguientes 3 archivos en nuestro folder:
- Dockerfile
- Gemfile
- Gemfile.lock

Creemos un archivo `docker-compose.yml`
```
web:
  build: .
  command: rails server -b 0.0.0.0
  ports:
    - '3000:3000'
  volumes:
    - '.:/usr/src/app'
```

Esto nos permitira iniciar y parar nuestro contenedor de `web`, támbien podremos ejecutar comandos en el contexto del contenedor. La sección de `build` le dice a Compose que este contenedor debe ser construido con el Dockerfile en este directorio. La sección de `command` define que comando debe ser ejecutado cuando el contenedor empieze. Queremos correr el comando de `rails server`.
La sección de `ports` le especifica que exponga el puerto 3000. La sección de `volumnes` le especifica que queremos montar nuestro directorio local `.` en `/usr/src/app`

Hagamos build de nuestra imagen
```
docker-compose build
```

Ahora si podemos usar rails
```
docker-compose run web rails new .
```

Nos va a preguntar si queremos que sobrescriba el Gemfile, le decimos que si (Y).

Tenemos que hacer unos cambios en el Gemfile para que funcione bien.

En la linea 15 descomentamos lo siguiente:
```
# gem 'therubyracer', platforms: :ruby
```

Y agregemos la gema `rails_12factor`, para que Rails se comporte mejor en un ambiente contenerizado.
```
gem 'rails_12factor'
```

Ahora que cambiamos el `Gemfile` tenemos que hacer `bundle install`

```
docker run -it -v $(pwd):/usr/src/app -w /usr/src/app ruby:2.2 bundle install && docker-compose build
```

Tomemos nota de la IP de boot2docker.
```
boot2docker ip
```

Corramos nuestra app de rails
```
docker-compose up
```

Ya podemos ingresar a nuestra app en nuestra ip:3000. ej. `http://192.168.59.104:3000/`

## Subamosle el nivel, 2 contenedores

Editemos el `Gemfile` para cambiar la gema de `sqlite3` a `pg`

Corramos `bundle install`
```
docker run -it -v $(pwd):/usr/src/app -w /usr/src/app ruby:2.2 bundle install && docker-compose build
```

Ahora necesitamos que postgres sea parte del stack de desarrollo. Replacemos nuestro `docker-compose.yml` con esto:
```
db:
  image: postgres:9.4
  environment:
    - POSTGRES_PASSWORD=asdfasdf

web:
  build: .
  command: rails server -b 0.0.0.0
  ports:
    - '3000:3000'
  volumes:
    - '.:/usr/src/app'
  links:
    - db
```

Lo que hicimos aqui fue configrar la variable de ambiente `POSTGRES_PASSWORD` para darle una contraseña. Támbien hicimos un cambio en `web` lo estamos conectando a la base de datos con `links`.

Cambiemos nuestro `database.yml` por uno para postgres.
```
default: &default
  adapter: postgresql
  timeout: 5000
  database: rails_development
  pool: 5
  host: db
  port: 5432
  username: postgres
  password: <%= ENV["DB_ENV_POSTGRES_PASSWORD"] %>

development:
  <<: *default

test:
  <<: *default
  database: rails_test

production:
  <<: *default
  database: rails_production
```

Iniciemos nuestro nuevo stack.
```
docker-compose up
```

Ahora creemos nuestra BD
```
docker-compose run web rake db:create
```

Hagamos un scaffold para que nuestra app haga algo...
```
docker-compose run web rails generate scaffold post title body:text published:boolean
```

Corremos las migrations
```
docker-compose run web rake db:migrate
```

Ya podemos ver nuestros posts!
```
http://192.168.59.104:3000/posts
```

Támbien puedes correr pruebas así:
```
docker-compose run web rake
```

## Producción

Hagamos login a `tutum`
```
docker login tutum.co
```

Lo que haremos ahora es un `Dockerfile` especifico para producción. Solo varia en unas pocas cosas. Primero correra el asset compailer de Rails, después configura la variable de ambiente `RAILS_ENV` y por ultimo definimos el commando que se correra por default.

Entonces hagamos un archivo llamado `Dockerfile.production`
```
FROM ruby:2.2-onbuild

RUN rake assets:precompile

ENV RAILS_ENV production

CMD ["rails", "server", "-b", "0.0.0.0"]
```

Támbien es buena practica crear un `.dockerignore` para prevenir que se suban archivos inecesarios al servidor, como puede ser el `.git` o la carpeta de `tmp/`

En este caso solamente agregmos el archivo con:
```
tmp/
```

Ahora ya podemos build un contenedor de producción que enviaremos a nuestro repositorio privado de Tulum.
```
docker build -t tutum.co/acrogenesis/rails -f Dockerfile.production .
```

Ahora ingresemos a https://dashboard.tutum.co/stack/ para crear un nuevo Stack.

Le ponemos el nombre que queramos, y esto es lo que usaremos como Stackfile:
```
db:
  image: postgres:9.4
  environment:
    - POSTGRES_PASSWORD=password_ULTRA_secreto

web:
  image: tutum.co/acrogenesis/rails
  ports:
    - '80:3000'
  links:
    - db
  environment:
    - SECRET_KEY_BASE=
```

Genermos un secret para nuestra app.
```
docker-compose run web rake secret
```

Ahora en `SECRET_KEY_BASE` copiemos el string que nos regreso
```
61b52dffb8fc7b27591ae52c65c6eb2e61f7b4706794388a29eec5fe7cfaf063ff78272480b30b52f0992f0dbb63985751f2958b5e614175e3cda1b2ee6b1fe2
```

Notaras que es casi identico a nuestro `docker-compose.yml`. La unica diferencia es que le pusimos un image a nuestro contenedor `web` y removimos la sección de `volumnes`. Demos click en `Create & Deploy`.

Una vez que nuestro `Stack` este corriendo lo podemos ver dando click en el tab de `Endpoints`, donde encontraremos el URL. Habramoslo en nuestro navegador.
Se vera algo así:
```
tcp://web.rails.acrogenesis.svc.tutum.io:80
```
Navegemos a nuestro servicio en producción
```
http://web.rails.acrogenesis.tutum.io:80/posts
```

Nos da un error!!!
Si regresamos al dashboard de tutum y navegamos hacia el stack, y le damos click en el sevicio de `web`. Damos click en el tab de `Logs`

Veremos el siguiente error:
```
ActiveRecord::NoDatabaseError (FATAL:  database "rails_production" does not exist
```

Claro! Ejecutemos `rake db:create` y `rake db:migrate`.

Con Tutum lo podemos hacer desde el dashboard.

Demos click en el tab de `Containers` en el servicio de `web`. Y demos click en `>_ Terminal`.

Eso abrira una terminal dentro de nuestro contenedor.

Simplemente ejecutemos el comando como lo hariamos noramalmente.

```
rake db:create && rake db:migrate
```
