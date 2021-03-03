# Curso de Docker en Platzi

Ésta es una aplicación de ejemplo para el curso de Docker de Platzi por Guido
Vilariño.

Encuentra más información en https://platzi.com, suscríbete al curso y aprende
a usar Docker de manera profesional.

# Mis apuntes

# Docker

[Docker training at Platzi](https://platzi.com/clases/2066-docker/) (by Guido Vilariño)

## How to install

[Docker Desktop](https://www.docker.com/products/docker-desktop)

## General

- [Print version information](https://docs.docker.com/engine/reference/commandline/cli/#docker)

    ```bash
    $ docker --version
    $ docker -v
    ```

## Containers

- Run a container

    `docker run <image-name>`
    
    e.g `docker run hello-world`
    
- List running containers

    `docker ps`

- List all the containers

    `docker ps -a`

- See details of a container

    `docker inspect <container-id | container-name>`
 
 - Run a container assigning a name to it

    `docker run --name <container-desired-name> <image-name>`

- Rename a container

    `docker rename <container-original-name> <container-new-name>`
    
- Remove an stopped container

    `docker rm <container-id | container-name>

- Force removing a container even if it's running

    `docker rm -f <container-id | container-name>

- Remove all the stopped containers

    `docker container prune`

- Run a container in interactive mode

    `docker run -it <image-name>`
    
    e.g `docker run -it ubuntu`

- Run a container in detached mode overriding the main process

    `docker run [-d | --detach] <container-name> <override-command>`
    
- Execute a command or process in a running container

    `docker exec -it <container-name> <command>`
    
The container will not stop until the MAIN process finishes.

- Stop a running container

    `docker stop <container-name | container-id>`

- Run a container in detached mode and exposing an interal port

    `docker run --name <container-desired-name> -d -p <host-port>:<container-port> <image-name>``

- See the logs of a running container

    `docker logs <container-name | container-id>`
    
    `-f (follow)`: see logs in real time in console
    `--tail [lines]`: only show the last lines of logs
    
## Data in Docker

### Bind mount (to share data between host and container)

    `docker run -d --name <container-name> -v <host-folder-to-folder-to-mount-in-container>:<container-folder> <image-name>`

    This gives access to the folder in the host
    
### Volumes

Improves security. It is used by a docker container and only that container can access to that folder.

- List all volumes

    `docker volume ls`
    
- Create a volume
    
    `docker volume create <volume-name>`

- Run a container and mount a volume that can be used by that container

    `docker run -d --name <container-name> --mount src=<volume-name>,dst=<container-folder> <image-name>`
    
    If `<volume-name>` doesn't exist, docker creates it

![](https://i1.wp.com/cdn-images-1.medium.com/max/800/1*bo6IOrBjaHbtkPgTKT08NA.png?w=1170&ssl=1)

### Insert and extract files from a container

- Copy a file or folder from host to container (it doesn't need the container to be running)

    `docker cp <relative-path-of-file/folder-to-be-copied> <container-name>:<destiny-inside-container>`

- Copy a file or folder from container to host (it doesn't need the container to be running)

    `docker cp <container-name>:<file/folder-inside-container> <relative-destiny-path>`
    
## Images

- List local images

    `docker image ls`

- Images are downloaded from hub.docker.com

- Download an image from an external repository (by default Dockerhub)

    `docker pull <image_name>:<tag>`

- Build our own image

    1. Create Dockerfile file (executed in build time) => Layers
    2. Build an image using our Dockerfile
    
    `docker build -t <base_image>:<tag_image_name> <Dockerfile context>`
    -t: tag
    docker build -t ubuntu_platzi .

    3. Run container from our image

    `docker run -it <base_image>:<tag_image_name> <command>`
    docker run -it ubuntu:platzi bash

- Publish an image

    1. Log in Dockerhub

    `docker login -u <username> -p <password>`

    2. Retag the image

    `docker tag <image>:<tag> <dockerhub_username>/<image>:<tag>`
    `docker tag ubuntu:platzi inigomarquinez/ubuntu:platzi`
    
    3. Publish the image

    `docker push <dockerhub_username>/<image>:<tag>`
    `docker push inigomarquinez/ubuntu:platzi`

- Show docker image layers
  
    1. Using command

    `docker history <image_name>:<tag>`

    eg. `docker history ubuntu:platzi`

    2. Using [dive tool](https://github.com/wagoodman/dive)

    `dive <image_name>:<tag>`

    eg. `dive ubuntu:platzi`

## Docker as a development tool

### Run a nodejs application inside a Docker container

1. Basic Dockerfile to run a nodejs application exposing a port (there must be a nodejs application in the folder from where we're running the build command)

        ```Dockerfile
        FROM node:12

        COPY [".", "/usr/src/"]

        WORKDIR /usr/src

        RUN npm install

        EXPOSE 3000

        CMD ["node", "index.js"]
        ```

2. To use that image, first build it

        `docker build platziapp .`

3. And then Run the container

        `docker run -p 3000:3000 platziapp`

### Using layer cache to optimize building images

        ```Dockerfile
        FROM node:12

        COPY ["package.json", "package-lock.json", "/usr/src/"]

        WORKDIR /usr/src

        RUN npm install

        # This won't copy package.json", "package-lock.json as they are not modified
        COPY [".", "/usr/src/"]

        EXPOSE 3000

        CMD ["node", "index.js"]
        ```

        This new Dockerfile will reuse the first layers (up to RUN npm install as long as we don't modify package.json or package-lock.json), so build time will be reduced, as it won't install the dependencies again.

- Update container whenever we change our code but without having to rebuild the complete image:

    1. Share our code with the contianer using bind mounts
    2. Use nodemon to detect changes in our code

        ```Dockerfile
        FROM node:12

        COPY ["package.json", "package-lock.json", "/usr/src/"]

        WORKDIR /usr/src

        RUN npm install

        # This won't copy package.json", "package-lock.json as they are not modified
        COPY [".", "/usr/src/"]

        EXPOSE 3000

        CMD ["npx", "nodemon", "index.js"]
        ```

        This command will copy all in source to destiny, hiding everything else. So if we copy a folder and it doesn't have dependencies installed, it will fail. This is why we onpy copy index.js in our example

        `docker run -p 3000:3000 -v <source>:<destiny>`

        eg `docker run -p 3000:3000 -v /Users/.../index.js:/usr/src/index.js platziapp`

### Docker networking

- List networks

    ```bash
    $ docker network ls
    ```

    Default networks in Docker: bridge, host, none

- Create a new network

    ```bash
    $ docker network create --attachable <name>
    ```

    `--attachable`: allows other containers to connect to this network

    `$ docker network create --attachable platzinet`

- Inspect an existing network

    ```bash
    $ docker network inspect <name>
    ```

    `$ docker network inspect platzinet`

- Connect a container to a network

    1. Container with database: 
    
    `$ docker run -d --name db mongo`

    2. Coonect to network

    ```bash
    $ docker network connect <network-name> <container-name>
    ```

    `$ docker network connect platzinet db`

    3. Run our app setting an env variable with connection string to database (if two container are in the same network, they can find each other by name)

    `$ docker run -d --name app -p 3000:3000 --env MONGO_URL=mongodb://<container-name>:27017/<database-name> <image-name>`

    `$ docker run -d --name app -p 3000:3000 --env MONGO_URL=mongodb://db:27017/test platziapp`

    `$ docker network connect platzinet app`

## Docker Compose

### Example

- `docker-compose.yml` file example

    ```yml
    # required
    version: "<version>"

    # Services used in our application (micro-services architecture): different components of our app to run correctly
    services:
        <service_1_name>:
            # imaged used by the container
            image: <image_1_name>
            # environment variables (= --env)
            environment:
                <env_name>: <env_value>
            # dependencies on other services
            depends_on:
                - <service_2_name>
            # ports exposed (= -p)
            ports:
                - "<host_port>:<container_port>"
        
        <service_2_name>:
            image: <image_2_name>
    ```

    ```yml
    version: "3.8"

    services:
        app:
            image: platziapp
            environment:
                MONGO_URL: "mongodb://db:27017/test"
            depends_on:
                - db
            ports:
                - "3000:3000"
        
        db:
            image: mongo
    ```

- Run it

    ```bash
    $ docker compose up -d
    ```

    `-d`: detached


------------

# 1️⃣ Introduction

## Bienvenida al curso

## Las tres áreas en el desarrollo de software profesional

## Virtualización

# 2️⃣ Containers

## Primeros pasos: hola mundo

## Conceptos fundamentales de Docker: contenedores

## Comprendiendo el estado de Docker

## El modo interactivo

## El ciclo de vida de un contenedor

## Exponiendo contenedores

# 3️⃣ Data in Docker

## Bind mounts

## Volúmenes

## Insertar y extraer archivos de un contenedor

# 4️⃣ Images

## Conceptos fundamentales de Docker: imágenes

## Construyendo una imagen propia

## El sistema de capas

# 5️⃣ Docker as a development tool

## Usando Docker para desarrollar aplicaciones

## Aprovechando el caché de capas para estructurar correctamente tus imágenes

## Docker networking: colaboración entre contenedores

# 6️⃣ Docker Compose

## Docker Compose: la herramienta todo en uno

## Subcomandos de Docker Compose

- See containers generated by docker compose

```bash
$ docker-compose ps
```

|Name|Command|State|Ports|
|---|---|---|---|
|<folder>_<service_name>_<index>|   |   |   |

- See logs

```bash
$ docker-compose logs
```

```bash
$ docker-compose logs <service_name_1> <service_name_2>
$ docker-compose logs app
```
Options:
- `-f`: follow logs

- Run a command in a container inside docker compose

```bash
$ docker-compose exec <service_name> <command>
$ docker-compose exec app bash
```

We don't need to put `-it` for interactive mode in docker compose

- Destroy all

```bash
$ docker-compose down
```

## Docker compose como herramienta de desarrollo

En vez de usar una imagen inmutable, podemos decirle a docker compose que cree una a partir de los archivos en disco

En vez de `image` usamos `build` y especificamos el contexto de build

    ```yml
    version: "3.8"

    services:
        app:
            build: .
            environment:
                MONGO_URL: "mongodb://db:27017/test"
            depends_on:
                - db
            ports:
                - "3000:3000"
        
        db:
            image: mongo
    ```

Entonces con `docker-compose build` hará un build de los servicios del docker-compose

Y para levantarlo `docker-compose up -d`

Para poder hacer cambios en nuestro editor de texto y que se aplicquen directamente sin tener que hacer un rebuild cada vez que hagamos cambios, podemos usar el bind mount.

Añadimos una nueva instrucción al docker-compose, `volumes` que define dónde montar mi código en la imagen docker.
Y otra para evitar que se sobreescriban los node_modules al copiar de local a la imagen

    ```yml
    version: "3.8"

    services:
        app:
            build: .
            environment:
                MONGO_URL: "mongodb://db:27017/test"
            depends_on:
                - db
            ports:
                - "3000:3000"
            volumes:
                - .:/usr/src
                - /usr/src/node_modules
        
        db:
            image: mongo
    ```

Y para regenerarlo: `docker-compose up -d`

Hay que hacer un paso extra para que node se de cuenta de que hay cambios en el código, sobreescribiendo el comando por defecto en el docker-compose para arrancar con nodemon, añadiendo `command`

    ```yml
    version: "3.8"

    services:
        app:
            build: .
            environment:
                MONGO_URL: "mongodb://db:27017/test"
            depends_on:
                - db
            ports:
                - "3000:3000"
            volumes:
                - .:/usr/src
                - /usr/src/node_modules
            command: npx nodemon index.js
        
        db:
            image: mongo
    ```

Y para regenerarlo: `docker-compose up -d`

## Compose en equipo: override

Al estar docker-compose normalmente versionado en nuestro repositorio, ¿cómo podemos hacer para modificarlo localmente y que no afecte a la versión del repositorio? Con una herramienta de docker-compose llamada *compose override*: es igual al compose file y que permite personalizar el compose solo para nuestro local.

1. Dejamos el docker-compose original

    ```yml
    version: "3.8"

    services:
        app:
            image: platziapp
            environment:
                MONGO_URL: "mongodb://db:27017/test"
            depends_on:
                - db
            ports:
                - "3000:3000"
        
        db:
            image: mongo
    ```

2. Y creamos un `docker-compose.override.yml` (detectado automáticamente por docker-compose) para sobreescribir algunas partes del docker-compose original. Hay cosas que se van a mergear bien (por ejemplo enbvironment), pero otras que no (por ejemplo, ports, que es mejor solo en el docker-compose original).

    ```yml
    version: "3.8"

    services:
        app:
            build: .
            volumes:
                - .:/usr/src
                - /usr/src/node_modules
            command: npx nodemon index.js
        
        db:
            image: mongo
    ```

- Escalar mi docker-compose a varias instancias:

`docker-compose up -d --scale app=2` => fallaría por que un puerto solo se puede abrir una vez, por lo que habría que definir un rango para solucionar este problema:

    ```yml
    version: "3.8"

    services:
        app:
            image: platziapp
            environment:
                MONGO_URL: "mongodb://db:27017/test"
            depends_on:
                - db
            ports:
                - "3000-3001:3000"
        
        db:
            image: mongo
    ```

Y lo levantamos con `docker-compose up -d`

Esto levantará dos contenedores `docker_app_1` y `docker_app_2`, y tendré un servicio corriendo en el puerto 3000 y otro en el puerto 3001, y ambos yendo al puerto 3000 del contenedor.

Esto permite probar programación concurrente o servicios a escala, con varios front-end a una misma base de datos.

`docker-compose down` para pararlo todo.

# 7️⃣ Docker advanced

## Administrando tu ambiente de Docker

- Gestión de recursos:

`$ docker ps -a` (veo todos los contenedores de mi máquina)
`$ docker ps -aq` (veo solo el id de todos los contenedores de mi máquina)
`$ docker container prune` (borra todos los contenedores inactivos)
`$ docker rm -f $(docker ps -aq)` (borra todos los contenedores que estén corriendo o apagados)
`$ docker network ls` (lista todas las redes)
`$ docker network prune` (borra todas las redes inactivas)
`$ docker volume ls` (lista todos los volumes)
`$ docker volume prune` (borra todos los volumes inactivos)
`$ docker image ls` (lista todas las imágenes)
`$ docker image rm -f $(docker image ls -aq)` (para borrar todas las imagenes existentes)
`$ docker system prune` (borra todo lo que no se esté usando)

- En cuanto a la gestión de los recursos a los que tienen acceso los contenedores cuando se están ejecutando:

`$ docker run -d --name app --memory 1g platziapp` (limito el uso de memoria)
`$ docker stats` (veo cuantos recursos consume docker en mi sistema)
`$ docker inspect app` (puedo ver si el proceso muere por falta de recursos => "State")

## Deteniendo contenedores correctamente: SHELL vs. EXEC

Ejemplo en el repo: /avanzado/loop
`$ docker build -t loop .` (construyo la imagen)
`$ docker run -d --name looper loop` (corro el contenedor)
`$ docker stop looper` (le envía la señal SIGTERM al contenedor)
`$ docker ps -l` (muestra el ps del último proceso)
`$ docker kill looper` (le envía la señal SIGKILL al contenedor)
`$ docker exec looper ps -ef` (veo los procesos del contenedor)

SHELL form => docker va a correr CMD como un comando HIJO del shell (cuyo PID va a ser distinto de 1, con lo cual SIGTERM no lo va a matar directamente)

    ```dockerfile
    FROM ubuntu:trusty
    COPY ["loop.sh", "/"]
    CMD /loop.sh
    ```

    $ docker exec looper ps -ef
    UID     PID     CMD
    root    1       /bin/sh -c /loop.sh
    root    7       bash /loop.sh

    $ docker stop looper
    No lo va a detener inmediatamente

    $docker exit -l
    STATUS => Exited (137)

The shell form prevents any CMD or RUN command line arguments from being used, but has the disadvantage that your ENTRYPOINT will be started as a subcommand of /bin/sh -c, which does not pass signals. This means that the executable will not be the container’s PID 1 - and will not receive Unix signals - so your executable will not receive a SIGTERM from docker stop <container>.

vs EXEC form => de esta forma sí va a correr el comando directamente (con PID = 1) => Graceful shutdown

    ```dockerfile
    FROM ubuntu:trusty
    COPY ["loop.sh", "/"]
    CMD [/loop.sh]
    ```

    $ docker exec looper ps -ef
    UID     PID     CMD
    root    1       bash /loop.sh

    $ docker stop looper
    Lo va a detener inmediatamente porque la señal SIGTERM llega directamente a mi proceso

    $docker exit -l
    STATUS => Exited (0)

Unlike the shell form, the exec form does not invoke a command shell. This means that normal shell processing does not happen. For example, RUN [ “echo”, “$HOME” ] will not do variable substitution on $HOME. If you want shell processing then either use the shell form or execute a shell directly, for example: RUN [ “sh”, “-c”, “echo $HOME” ]. When using the exec form and executing a shell directly, as in the case for the shell form, it is the shell that is doing the environment variable expansion, not docker.

## Contenedores ejecutables: ENTRYPOINT vs CMD

Repo: /avanzado/ping

Contenedor que acepte parámetros por línea de comandos => ENTRYPOINT: comando que se va a ejecutar siempre, salvo que se haga un override, y usando como parámetro lo que ponga en CMD

    ```dockerfile
    FROM ubuntu:trusty
    ENTRYPOINT ["/bin/ping", "-c", "3"]
    CMD ["localhost"]
    ```

    $ docker run --name pinger ping google.com => ahora ping a google

    $docker ps -l
    CONTAINER ID        COMMAND
    xxx                 "/bin/ping -c 3 google.com" => combinación de ENTRYPOINT y CMD

There can only be one CMD instruction in a Dockerfile. If you list more than one CMD then only the last CMD will take effect.
The main purpose of a CMD is to provide defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.
If CMD is used to provide default arguments for the ENTRYPOINT instruction, both the CMD and ENTRYPOINT instructions should be specified with the JSON array format.

## El contexto de build

https://docs.docker.com/engine/reference/builder/#dockerignore-file

Fichero `.dockerignore` en el que se especifican qué archivos y carpetas NO hay que considerar en el contexto de build

`$ docker build -t prueba .` (creo la imagen)
`$ docker run -d --rm --name app prueba` (corro el contenedor)
en el archivo .dockerignore puedo poner todo lo que no quiero que copie del contexto de build
`$ docker exec -it app bash` (entro al contenedor y verifico que no se haya copiado lo que está en el .dockerignore)

## Multi-stage build



## Docker-in-Docker



# 8️⃣ Close

## Cierre del curso

