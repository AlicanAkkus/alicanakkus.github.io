---
layout: post
title: Docker Compose
permalink: /blog/docker/docker-compose
summary:  This blog explains the purpose of Docker compose build and provide examples of how they help us generate smaller and more efficient Docker Containers.
---

This blog explains the purpose of Docker compose build and provide examples of how they help us generate smaller and more efficient Docker Containers.

After you completed this blog you will be able to understand basic of docker compose and how use it.

### Docker Compose
In terms, Docker compose that means as the below;
> Compose is a tool for defining and running complex applications with Docker. With Compose, you define a multi-container application in a single file, then spin your application up in a single command which does everything that needs to be done to get it running.

Docker Compose is a tool for defining and running multi-container Docker applications. It's just that.

With Docker compose, we will run more than one containers together and dependent other one. For example, you can define a container for database, web server and cache with depend on db container. In this example we will use the redis.

> Note : Docker Compose uses the same API used by other Docker commands((CLI commands)) and tools.

Let's begin;

We will use the docker-compose.yml configuration file for creating application's services. It's simple yaml file. After creating compose configuration file, we will start all the services from configuration with only one command.  

Docker compose can be used in many different requirement and/or in many different ways.
1. Development environments
2. Automated testing environments
3. Single host deployments

Firstly learn the version of docker compose on your docker installation as below;
![docker compose version](/images/docker/docker-compose-version.png)

And then listing docker images which installed on docker.
![docker compose images](/images/docker/docker-compose-image-list.png)
> Docker compose was already installed on docker installation.

We have four images. We will use all of them in our docker compose example.

#### Compose file
Let's begin to create the compose file.
**docker-compose.yml**
```
version: '2' #(1)

#(2)
services:
    server_a:
        image: nginx:latest
        ports:
            - "8001:80"

    server_b:
        image: nginx:latest
        ports:
            - "8002:80"

    server_c:
        image: nginx:latest
        ports:
            - "8003:80"
```
We've defined sample configuration file. It's contains **version** attribute and service definitions. We can define service at the **services** section. **server_?** is the alias name of the service. Image attr is that means which image will be run and ports definitions for the docker container between host.

**version** means that what version of the docker compose. We've using version **2**. We will start the all service only via **docker-compose up** command. Go to docker-compose.yml file directory and run it;

![docker compose up](/images/docker/docker-compose-up.png)

Starting three nginx server and go to localhost:8001 and etc. You can find that nginx up!

If you want to start the docker-compose with detached mode you can do it by giving command;
```
docker-compose up -d
```
Let's check the status of containers;
```
iyzi-it125:docker alican.akkus$ docker-compose ps
      Name                Command          State          Ports         
-----------------------------------------------------------------------
docker_server_a_1   nginx -g daemon off;   Up      0.0.0.0:8001->80/tcp
docker_server_b_1   nginx -g daemon off;   Up      0.0.0.0:8002->80/tcp
docker_server_c_1   nginx -g daemon off;   Up      0.0.0.0:8003->80/tcp
iyzi-it125:docker alican.akkus$
```
> Please note that, you can use the all core docker commands with docker-compose such as above.

Containers log can be seen as below;
```
iyzi-it125:docker alican.akkus$ docker-compose logs
Attaching to docker_server_a_1, docker_server_c_1, docker_server_b_1
server_a_1  | 172.19.0.1 - - [15/May/2017:18:13:34 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.81 Safari/537.36 OPR/45.0.2552.635" "-"
server_a_1  | 2017/05/15 18:13:34 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.19.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8001", referrer: "http://localhost:8001/"
server_a_1  | 172.19.0.1 - - [15/May/2017:18:13:34 +0000] "GET /favicon.ico HTTP/1.1" 404 571 "http://localhost:8001/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.81 Safari/537.36 OPR/45.0.2552.635" "-"
iyzi-it125:docker alican.akkus$
```

You can stop docker-compose with below command;
```
iyzi-it125:docker alican.akkus$ docker-compose stop
Stopping docker_server_a_1 ... done
Stopping docker_server_c_1 ... done
Stopping docker_server_b_1 ... done
iyzi-it125:docker alican.akkus$
```

Shutdown/Clean up;
* **docker-compose down** : will remove the containers.
* **docker-compose down --volumes** : will remove the containers and volumes.

Note : You can see all docker-compose command list by typing **docker-compose** and output shows as;
```
iyzi-it125:docker alican.akkus$ docker-compose
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name (default: directory name)
  --verbose                   Show more output
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the name specified
                              in the client certificate (for example if your docker host
                              is an IP address)

Commands:
  build              Build or rebuild services
  bundle             Generate a Docker bundle from the Compose file
  config             Validate and view the compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
iyzi-it125:docker alican.akkus$
```

> Note: All of the **docker-compose** commands may be not running correctly because yaml file is placed the other directory.

İf you want to remove all container you can use the this command;
```
docker-compose stop
```
Sometimes we don't want to run all containers from docker-compose.yml configuration file. We will use partial start as follow syntax -> **docker-compose up service_name other_service_name** for example;
```
iyzi-it125:docker alican.akkus$ docker-compose up server_a server_b
Creating docker_server_a_1
Creating docker_server_b_1
Attaching to docker_server_a_1, docker_server_b_1
```
Let's connect to container with exec command;
```
iyzi-it125:docker alican.akkus$ docker-compose up -d server_a server_b
Starting docker_server_a_1
Starting docker_server_b_1
iyzi-it125:docker alican.akkus$ docker-compose exec server_a /bin/bash
root@b9bb8ced17be:/# uname -a
Linux b9bb8ced17be 4.9.27-moby #1 SMP Thu May 11 04:01:18 UTC 2017 x86_64 GNU/Linux
root@b9bb8ced17be:/#
```

All containers could connect to other via alias name. For example we will use the ping to other server;
```
iyzi-it125:docker alican.akkus$ docker-compose exec server_a ping -c 3 server_b
PING server_b (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: icmp_seq=0 ttl=64 time=0.761 ms
64 bytes from 172.19.0.3: icmp_seq=1 ttl=64 time=0.222 ms
64 bytes from 172.19.0.3: icmp_seq=2 ttl=64 time=0.164 ms
--- server_b ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.164/0.382/0.761/0.269 ms
iyzi-it125:docker alican.akkus$
```

Some containers depends with other one. For example, web application depended with db and cache.
Create the new configuration file as below;
```
version: '2' #(1)

#(2)
services:
    db:
        image: mysql:latest
        volumes:
         - db_data:/var/lib/mysql
        restart: always
        environment:
         MYSQL_ROOT_PASSWORD: 1234
         MYSQL_DATABASE: challenge
         MYSQL_USER: root
         MYSQL_PASSWORD: 1234
        ports:
            - "3306:3306"
    app:
        image: aakkus/iyzico-challenge
        ports:
            - "8080:8080"
        depends_on:
            - db
            - redis

    redis:
        image: redis:latest
        ports:
            - "6379:6379"
volumes:
    db_data:
```

Outputs looks like below;

```
iyzi-it125:docker alican.akkus$ docker-compose up -d
WARNING: Found orphan containers (docker_server_b_1, docker_server_a_1) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
Creating docker_db_1
Creating docker_redis_1
Creating docker_app_1
iyzi-it125:docker alican.akkus$
```

**app** service depended to db and redis. Docker engine, firstly run the depended containers and then run the others.

I hope this blog is useful for you. See you next posts.

> Alican Akkus.
