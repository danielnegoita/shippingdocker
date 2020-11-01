## Running example and Docker commands extracted from [Server for hackers](https://serversforhackers.com/t/containers) free  series on containers. 

### 

### # Docker commands

**[Docker CLI documentation](https://docs.docker.com/engine/reference/commandline/docker/)**

What are **images** and **containers** in Docker?
To make an analogy with PHP:
- **images** are like __classes__ (ex: `SomeClass.php`) 
- **containers** are like __objects__ (ex: `$container = new SomeClass()`)

__Notes:__
Make sure that the scripts inside a container does not run as a __daemon__ (in the background), keep them in the foreground for Docker to see them and debug (ex: nginx, php).

###

**List all `commands` and `args` availabe**
```sh
$ docker --help
```

**Show available `args` for a `command` (ex: ps, images)**
```sh
$ docker <command-name> --help
```

**List all images**
```sh
$ docker image ls
```

**List only the images `ids`**
```sh
$ docker image ls -q
```

**Remove an image**
```sh
$ docker image rm <image-name>:<tag-name>
```

**Remove all images**
`$(docker image ls -q)` - returns a complete list with all the containers **ids** 
```sh
$ docker image rm $(docker image ls -q)
```

**List all containers (including the ones not running)**
`-a` - show all the containers
```sh
$ docker ps -a
```

**List all __running__ containers**
```sh
$ docker ps
```

**Stops container with id `<container-id>` (ex: as5335fsa639ggf)**
It will also remove it if the container has been started with `-rm` flag
```sh
$ docker stop <container-id>
```

**Remove container with id `<container-id>` (ex: as5335fsa639ggf)**
```sh
$ docker rm <container-id>
```


**Create a container from an image**\
`--rm` - destroy the container after is done running\
`-d` - run the container in the background\
`-v $(pwd)/<directory-name>:/var/ww/html` - share (symlink/mount/bind) a directory. It shares `<directory-name>` from the current directory (`$(pwd)`) and place it _in the docker container_ at `/var/ww/html`\
`-p 8080:80` - share port `8080` from _localhost_ to port `80` inside of the container\
`--name` - give the container a name\
`- e <enviroment-variable-name>` - pass a gloabal enviroment variable to the container\
`--network=<network-name>` - join the network that we created called `<network-name>` (ex: _appnet_)\
`-w <path-to/directory-name>` - set the current working directory. So, the container will spin up, will `cd` into this directory and it will run the `<command(s)>` appended\
`<image-name>` - the image name from which to create the container (ex: rabbitmq, mailhog/mailhog, php)\
`<command> <args...> (optional)` - run this command inside the container after has run (ex: `nginx`, `nginx -g 'daemon off;'`)
```sh
# Note: replace -d with -it to output logs in the console.
$ docker run --rm -d -v $(pwd)/<directory-name>:/var/ww/html -p 8080:80 <image-name>:<image-tag> <command> <args...>

# or on multipe lines.
$ docker run --rm -d \
    -v $(pwd):/application:/var/www/html \
    -p 8080:80 \
    <image-name>:<image-tag> <command> <args...>
    
# example 2: install a project via composer 
$ docker run --rm -d -p 8080:80 -v $(pwd):/var/ww/html -w /var/ww/html <image-name>:<image-tag> composer create-project laravel/laravel <directory-name>
    
# Give the container a name
$ docker run ... --name=mydb

# Pass enviroment variables to the container
$ docker run ... --name=my_db -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=my_datatabse_name

# Join a network ('my_db' will be the container name inside the network)
$ docker run ... --name=my_db --network=appnet
```

**Create a new container and run a _service_ in it**\
`--rm` - destroy the container after is done running.\
`-it` - interact with the container (is just like I would `ssh` into the container...but is not actual _ssh_)\
`<image-name>` - the image name from which to create the container (ex: ubuntu:18.04)\
`<service-name>` - which service to run in the container after it has started (ex: `bash` - open _bash_ inside the container, `nginx` - run _nginx_ inside the container etc.)\
`-d` - run the container in the background
```sh
# example of service names: bash, php, nginx
$ docker run --rm -it <image-name> <service-name>

# example 1: running a container in the background and start nginx 
$ docker run -d <image-name> nginx
```

**Execute a command inside a _running_ container**\
`-it` - interact with the container (is just like I would `ssh` into the container...but is not actual _ssh_)\
`<container-id>` (or `<host-name>` ex: my_db) - the id of the _running_ container. Get it with `docker ps` (ex: 91ec754c8351)\
`<command>` - the command to run inside the container (ex: `bash`, `cat`)\
`-w <path-to/directory>` - execute a command in the specified directory (ex: _/var/www/html_)
```sh
$ docker exec -it <container-id> <command>

# if we give the container a name (ex: --name=my_db) we can replace <container-id> with the <host-name> (ex: my_db)
$ docker exec -it <host-name> <command>

# execute a command in a directory inside the container
$ docker exec -it -w <path-to/directory> <host-name> <command>

#example: 'app' is the <host-name> and `php artisan list` is the command
$ docker exec -it -w /var/html/www app php artisan list
```

**Show the modifications made to a container from the base image**\
ex: If I create a container from `ubuntu:18.04` image, then I login into it and install `apache`, if I run the command it will show me that I installed `apache`
```sh
$ docker diff <container-id>
```

**Create a new _image_ from a _container_ with the `commit` command**\
Example: If I installed some packages in a _container_ (ex: apache, php, yarn etc.) and I want to _save them_, I `commit` the changes and a new **image** will be created.
Of course, that **image** will show up in the images list (`$ docker image ls`) and from it I can spin up a new **container**, identical to the initial one.\
`-a "<author-name>"` - the author of the commit (ex: "John doe")\
`-m "<message-body>"` - a commit message (ex: "Installed apache")\
`<container-id>` - the **id** of the container to commit (ex: as5335fsa639ggf)\
`<new-image-name>:<tag-name>` - the name of the new image that will be created (ex: myapache, myapache:latest, myapache:1.0). Note: _latest_ and _1.0_ are called image tags. If no tag is specified _latest_ will be added by default.
```sh
$ docker commit -a "<author-name>" -m "<message-body>" <container-id> <new-image-name>:<tag-name>
```

**Create a new _image_ from a _Dockerfile_ with the `build` command**\
Is similar to how we commit changes to a container to make a new image (see above)\
`-t <new-image-name>:<tag-name>` - name and tag of the new image (ex: myapp, myapp:latest, myapp/nginx:latest). Note _myapp/nginx_ represents the app name (namespaced name) and them the image name\
`-f <path-to-dockerfile>` - name of the Dockerfile (Default is 'PATH/Dockerfile') (ex: docker/nginx/Dockerfile)\
`<dockerfile-context>` - the context in which the Dockerfile to run (from where does _Dockerfile_ gets it's resources described in his content) (ex: docker/nginx)
```sh
$ docker build -t <new-image-name>:<tag-name>
    -f <path-to-dockerfile>
    <dockerfile-context>
```

**Read Docker logs output**\
`-f`(follow) - pass this flag to see the log output in real-time (when new output gets sent to the logs)\
`<container-id`> - the _id_ of the container from which to output the logs (get it by running `docker ps` command)\

How it works?\
If a process (nginx, supervisor, php etc.) running inside a container outputs to `/dev/stdout` or `/dev/stderr`, that output is sent to the Docker logging mechanism. So, the command will output the content of the Docker logging mechanism populated with the logs from `/dev/stdout` and `/dev/stderr`.\
Note: We need to configure each process (nginx, supervisor, php etc.) to output to `/dev/stdout` and `/dev/stderr`.
```sh
# basic usage
$ docker logs <container-id>

# see the container logs output in real-time by passing the "-f" (follow) flag
$ docker logs -f <container-id>
```

### # Docker `network` - [video](https://serversforhackers.com/c/div-docker-networks-intro)
For two, or more, containers to communicate with each other, they need to be part of the same network. For example the _app_ container which has [php](https://hub.docker.com/_/php) and [nginx](https://hub.docker.com/_/nginx) with the _[mysql](https://hub.docker.com/_/mysql)_ container with the DB. 

**List all networks**\
Note: By default Docker has 3 networks defined (bridge, host and none)
```sh
$ docker network ls
```

**Create a network [@docs](https://docs.docker.com/engine/reference/commandline/network_create/)**\
`<name>` - the network name (ex: appnet)\

Note: the default network type will be `bridge`. See [@docs](https://docs.docker.com/engine/reference/commandline/network_create/) for more details
```sh
$ docker network create <name> 
```

**Inspect a network [@docs](https://docs.docker.com/engine/reference/commandline/network_inspect/)**\
`<name>` - the network name (ex: appnet)
```sh
$ docker network inspect <name> 
```

**Remove a network**\
`<name>` - the network name (ex: appnet)
```sh
$ docker network rm <name> 
```

### # Docker `volume` - [video](https://serversforhackers.com/c/div-docker-volumes)

**List all volumes**
```sh
$ docker volume ls
```

**Create a new volume**\
`<name>` - the volume name (ex: _mydata_)
```sh
$ docker volume create <name>
```

**Remove a volume**\
`<name>` - the volume name (ex: _mydata_)
```sh
$ docker volume rm <name>

# remove all volumes
$ docker volume rm $(docker volume ls -q)
```

**How to avoid losing data, with Docker `volume`, after a container has been destroyed**\

If we create and run a container off an image (ex: a myslq 5.7 container) and then we destroy the container, we will lost all the data saved in the mysql databases.

To avoid this, we can _save_ the container data (state) in a `volume` (ex: mydata), so, when we recreate the container later on and _bind_ it to our volume (ex: mydata), all the information will be there, not lost anymore.

Note: `/var/lib/mysql` - is the folder where _mysql_ stores the data
```sh
# create 'mydata' volume
$ docker volume create mydata
# bind 'mydata' volume to the container data directory ('/var/lib/mysql' in our case)
$ docker run .... -v mydata:/var/lib/mysql ...
```


### # Docker `Dockerfile`
**`CMD` - Run a command inside the container after it has been started (on `docker run ...`)**

Why would we add it to Dockerfile? An option would have been to pass the `<command>` to run directly to `docker run ... <command>`, but by adding it in the `Dockerfile` it is not needed anymore and it will be runned automaticly.

Note: The command can be _overwritten_ by passing to the command `docker run ... <another-command>` another command `<another-command>`. In this case, our command from insede the `Dockerfile` _will NOT be runned anymore_!
```sh
CMD ["<command>"]

# Example - this will run "supervisord"
CMD ["supervisord"]
```

**`ENTRYPOINT` - Run a command or multiple commands inside a container using a bash/shell script - [video](https://serversforhackers.com/c/div-entrypoint-vs-cmd)**

Note: The `ENTRYPOINT` will be used _every time_ when we run a container, is not like a `CMD` which can be overwritten (see `CMD` details above)

How to example:
1. Define a bash/shell script `start-container.sh` on our localhost
2. In `Dockerfile` add a new line to `ADD/COPY` `start-container.sh` inside the container from out localhost (ex: `ADD start-container.sh /usr/bin/start-container`)
3. In `Dockerfile` add a `RUN` command to set permissions for read/write the file in the container (ex: `RUN chmod +x /usr/bin/start-container`)
4. Final step, add `ENTRYPOINT ["<path-to/bash-script-inside-the-container>"]` to `Dockerfile`

`<path-to/bash-script-inside-the-container>` - the _path_ (FROM INSIDE the container) to the script with all the commands to run inside that container
```sh
ENTRYPOINT ["<path-to/bash-script-inside-the-container>"]

# Example: "start-container" is the name of the file inside the container NOT in our localhost
ENTRYPOINT ["start-container"]

############ Example of "start-container" script

## Note: To get 'bash' path, access the container (docker exec <container-id>) and run the command `which bash`
#!/usr/bin/env bash
##
# Ensure /.composer exists and is writable
#
if [ ! -d /.composer ]; then
    mkdir /.composer
fi

chmod -R ugo+rw /.composer

##
# Run a command or start supervisord
#
if [ $# -gt 0 ];then
    # If we passed a command, run it
    exec "$@"
else
    # Otherwise start supervisord
    /usr/bin/supervisord
fi
```

#
##### `Dockerfile` example - [best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- `FROM <image-name>` - creates a layer from the <image-name> Docker image (ex: ubuntu:18.04)\
- `LABEL <key>="<value>"` - add labels to your image to help organize images by project, record licensing information, to aid in automation, or for other reasons\
- `ENV <VAR_NAME>=<value>` - environment variables specific to services you wish to containerize (ex: ADMIN_USER="john", APP_VERSION=9.3.4, DEBIAN_FRONTEND=noninteractive - this var will be used inside Ubuntu config on install...is like setting a global var)\
- `RUN <command(s)>` - executes command(s) in a new layer and creates a new image. It is often used for installing software packages. (ex: RUN apt-get update && apt-get install -y)\
- `ADD <path-to/local-file> <path-to/container-file>` - add a file from local directory to container provided path\
- `COPY <path-to/local-file> <path-to/container-file>` - the same as ADD. Copy the file from local directory to the container. Note: COPY is preferred over ADD (see best practices)\
- `CMD [<command>, <parameter(s)>]` - sets default command and/or parameters, which can be overwritten from command line when docker container runs (ex: CMD ["php", "-a"]). Note: `<parameter(s)>` is optional (ex: `CMD ["supervisord"]` - start _supervisord_ by default after the container starts running)
```sh
FROM ubuntu:18.04

LABEL maintainer="Chris Fidao"

ENV DEBIAN_FRONTEND=noninteractive
ENV ADMIN_USER="john"

RUN apt-get update \
    && apt-get install -y gnupg tzdata \
    ...

RUN apt-get update \
    && apt-get install -y curl zip unzip git supervisor sqlite3 \
       nginx php7.2-fpm php7.2-cli \
       ...
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && mkdir /run/php \
   ...
   
RUN echo $ADMIN_USER > ./john

ADD default /etc/nginx/sites-available/default
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["supervisord"]
```




###






### # `docker-compose` - [videos](https://serversforhackers.com/s/docker-in-dev-v2-ii) 

Require a variable inside a `docker-compose.yml` file like this: `${<VAR_NAME>}` (ex: `${APP_NAME}`).\
The variable can be set globally:
1. with the command `export <VAR_NAME>=value` and the run any `docker-compose` command
2. **OR** it can be set in an `.env` file, _located at the same level_ as `docker-compose.yml`

**List all (_only_) the containers created from `docker-compose.yml`**
```sh
$ docker-compose ps -a
```

**List all (_only_) the _active_ containers created from `docker-compose.yml`**
```sh
$ docker-compose ps
```

**Start all containers**\
`<args...> (optional)` - pass optional argumets (ex: `--help`, `-d` etc.)\

Note: As an workflow, is recommanded to use `up` and `down` commands, NOT `start` and `stop`
```sh
$ docker-compose up <args...>
```

**Stop and destroy all running containers**\
Note: As an workflow, is recommanded to use `up` and `down` commands, NOT `start` and `stop`
```sh
$ docker-compose down
```

**Restart all containers**\
Note: It will reset and regrab the new settings from `docker-compose.yml`. In case I did changes to the composer file it will apply them. 
```sh
$ docker-compose restart
```

**Executea a command inside a container**\
Note: If we didn't set a working directory (`working_dir`) in `docker-compose.yml` file for our application container, we will need to specify it in the command like this: `-w <path-to/working-directory>` (ex: `-w /var/www/html`)\
`<container-name>` - name of the container. Get it with command `docker-compose ps`\
`<command>` - the command you want to run inside the container (ex: `bash`, `php artisan migrate`, `composer require ...`)
```sh
$ docker-compose exec <container-name> <command>
# example including setting the working directory
$ docker-compose exec -w /var/www/html app composer require predis/predis
```

##### `docker-compose.yml` example
```sh
version: '3'
services:
  app:
    # If the image doesn't exsit on Docker Hub we need to add a 'build' parameter. 'build' + 'image' is the same as 'docker build -t shippingdocker/app:latest -f docker/app/Dockerfile docker/app'
    build:
      context: ./docker/app
      dockerfile: Dockerfile
    image: shippingdocker/app:latest
    networks:
      - appnet
    volumes:
      # "." is the path to the laravel application. In our case docker-compose.yml is in the same directory with the laravel application 
      - .:/var/www/html
    ports:
      - ${APP_PORT}:80
    working_dir: /var/www/html
  cache:
    image: redis:alpine
    networks:
      - appnet
    volumes:
      # see docs - https://hub.docker.com/_/redis and here https://github.com/docker-library/redis/blob/7ccc22760cc9b659916678a52654be8f43757551/6.0/alpine/Dockerfile
      - cachedata:/data
    ports:
      - 6379:6379
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: homestead
      MYSQL_USER: homestead
      MYSQL_PASSWORD: secret
    networks:
      - appnet
    volumes:
      # more details in the docs - https://hub.docker.com/_/mysql
      - dbdata:/var/lib/mysql
    ports:
      - ${DB_PORT}:3306
  node:
    build:
      context: ./docker/node
      dockerfile: Dockerfile
    image: shippingdocker/node:latest
    networks:
      - appnet
    volumes:
      # "." is the path to the laravel application. In our case docker-compose.yml is in the same directory with the laravel application 
      - .:/opt
    working_dir: /opt
    command: echo hi
networks:
  appnet:
    driver: bridge
volumes:
  dbdata:
    driver: local
  cachedata:
    driver: local
```



###





### # Linux commands [geeksforgeeks.org](https://www.geeksforgeeks.org/linux-commands/)

**`pwd`(print working directory) - Print path of the current working directory**
```sh
$ pwd
# Use it in docker commands like a dynamic variable, to share for example a directory into a docker container
$ docker run -d -p 8080:80 -v $(pwd)/myapplication:/var/www/html/public myimage:latest ...
```

**`which` - Locate the executable file associated with the given command by searching it in the path environment variable**\
`<name>` - can be one or multiple names
```sh
$ which <name>
# example 
$ which nginx, php, redis
```

**`ps`([Process Status](https://www.computernetworkingnotes.com/linux-tutorials/ps-aux-command-and-ps-command-explained.html)) - show the processes running under the logged in user account from the current terminal**\
`a` - prints the running processes from all users\
`u` - shows user or owner column in output\
`x` - prints the processes those have not been executed from the terminal\
`aux` (all the above options together) - print all the running processes in the system regardless from where they have been executed
```sh
$ ps aux
```

**`getent` - get entries from administrative database (ex: hosts, passwd, group, service, protocols etc.) - [@docs](https://www.unixtutorial.org/commands/getent)**
```sh
$ getent <administrative-database> <name>

# example 1: get the IPs a hostname (mysql) points to.
$ getent hosts mysql

# example 2: without parameters, this will show you all the groups found on the server
$ getent group
```


### # Mysql commands

How to create MySQL users accounts and grant privileges - [@see docs](https://linuxize.com/post/how-to-create-mysql-user-accounts-and-grant-privileges/)

How to show a list of all databases - [@see docs](https://linuxize.com/post/how-to-show-databases-in-mysql/)

How to show tables in a database - [@see docs](https://linuxize.com/post/show-tables-in-mysql-database/)
