---
layout: post
title: Docker Compose (Türkçe)
permalink: /blog/docker/docker-compose
summary: Bu yazıda Docker compose ile containerların build edilmesi, yönetilmesi ve efektik kullanılmasını sağladığımızı göreceğiz.
---

Bu yazıda Docker compose ile containerların build edilmesi, yönetilmesi ve efektik kullanılmasını sağladığımızı göreceğiz.
Bu yazıyı tamamladıktan sonra temel docker compose mantığını kavramış ve localinizde docker compose örneklerini kullanabiliyor olacaksınız.


### Docker Compose
Dokcer compose teknik olarak şöyle ifade edilir;
> Docker Compose, kompleks uygulamaların tanımlanmasını ve çalıştırılmasını sağlayan bir Docker tooludur. Docker compose ile birlikte birden fazla container tanımını tek bir dosyada yapabilir, tek bir komut ile uygulamanızın ihtiyaç duyduğu tüm gereksinimleri ayağa kaldırarak uygulamayı çalıştırabilirsiniz.

Docker compose, birden fazla container tanımlama ve çalıştırmak için kullanılır. Basitçe ifade böyle ifade edebiliriz.

Docker compose ile birden fazla container çalıştırabilir, bu containerlardan bazılarının birbirine depent durumda bağımlı kalmasını isteyebiliriz. Örneğin bir wordpress ayağa kaldırmak istiyorsunuz. Bu durumda bir mysql ve wordpress image tanımı yapabilir, wordpress'i db'ye depent edebilirsiniz. Bu sayede ilk olarak db ayağa kalkar sonra wordpress çalıştırılır. Böyle güzel yanları var. Biz örneğimizde Redis kullanacağız bu amaç için.

> Not : Docker Compose'da diğer Docker CLI komutlarını da rahatça kullanabilirsiniz.

Başlayalım;

Compose ile çalışabilmek için özel bir konfigürasyon dosyasına ihtiyacımız var. Yaml formatında olan bir dosya oluşturmalıyız. Adının **docker-compose.yml** olması zorunlu. Dosyamızı oluşturduktan sonra tek bir komut ile containerlarımızı ayağa kaldırabileceğiz.

Docker compose farklı ihtiyaç ve amaçlar için kullanılabilir.
1. Development environments : Şirkete yeni bir arkadaş geldi diyelim. Compose ile çok çok kısa sürede geliştirme yapabileceği bir ortam kendisine hazırlayabiliriz.
2. Automated testing environments : CI pipeline için kullanılabilir.
3. Single host deployments : Tek bir host üzerinde çalışsın herşey diyebiliriz.

İlk önce makinada kurulur olan compose versiyonunu öğrenelim;
![docker compose version](/images/docker/docker-compose-version.png)

Docker engine üzerinde kurulu imagelere bakalım;
![docker compose images](/images/docker/docker-compose-image-list.png)
> Not : Eğer güncel yada güncele yakın bir docker kurulu ise makinanızda Compose da yüklüdür default olarak.

Toplamda 4 adet image var. Örneğimizde hepsini kullanmaya çalışacağız.

#### Compose file
**docker-compose.yml** dosyamıza bakalım;
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
Örnek bir compose dosyası oluşturduk. Dosyamız içerisinde kullandığımız **version** ve **service** tanımları var. **server_?**  kısımları servislere verdiğimiz alias isimlendirmelerdir. Örneğimizde 3 adet nginx tanımı yaptık. Bunlara isimler verdik ve port tanımlarını yaptık.

Artık çalıştırmaya başlayabiliriz. Sadece **docker-compose up** komutunu verelim ve sonucu görelim.

![docker compose up](/images/docker/docker-compose-up.png)

3 adet nginx server ayağa kalktı. Local de 8001, 8002 ve 8003 portlarına giderek kontrol edebiliriz.

**docker-compose up** komutu compose ile console ataçlanmış/bağlanmış şekilde çalıştırır. Arka planda çalıştırmak için detached mode ile çalıştırılmalıdır. Şöyle ki;

```
docker-compose up -d
```
Containerların durumunu kontrol edelim;

```
iyzi-it125:docker alican.akkus$ docker-compose ps
      Name                Command          State          Ports         
-----------------------------------------------------------------------
docker_server_a_1   nginx -g daemon off;   Up      0.0.0.0:8001->80/tcp
docker_server_b_1   nginx -g daemon off;   Up      0.0.0.0:8002->80/tcp
docker_server_c_1   nginx -g daemon off;   Up      0.0.0.0:8003->80/tcp
iyzi-it125:docker alican.akkus$
```
> Not : Docker da bulunun core komutların neredeyse tamamını compose ile beraber kullanabiliriz yukardaki gibi.

Container loglarına gidelim. Örneğimizde pek gerek yok ama gerçek bir uygulama çalıştırıyor isek loglarını kontrol etmeyi isteyebiliriz;

```
iyzi-it125:docker alican.akkus$ docker-compose logs
Attaching to docker_server_a_1, docker_server_c_1, docker_server_b_1
server_a_1  | 172.19.0.1 - - [15/May/2017:18:13:34 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.81 Safari/537.36 OPR/45.0.2552.635" "-"
server_a_1  | 2017/05/15 18:13:34 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.19.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8001", referrer: "http://localhost:8001/"
server_a_1  | 172.19.0.1 - - [15/May/2017:18:13:34 +0000] "GET /favicon.ico HTTP/1.1" 404 571 "http://localhost:8001/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.81 Safari/537.36 OPR/45.0.2552.635" "-"
iyzi-it125:docker alican.akkus$
```

Containerları durdurmayı deneyelim;
```
iyzi-it125:docker alican.akkus$ docker-compose stop
Stopping docker_server_a_1 ... done
Stopping docker_server_c_1 ... done
Stopping docker_server_b_1 ... done
iyzi-it125:docker alican.akkus$
```

Shutdown/Clean up;
* **docker-compose down** : Containerları sil.
* **docker-compose down --volumes** : Containerları volumeleri ile birlikte kaldır. Bu aşamadan sonra containerın datalarına ulaşamayacağız.

Not : compose da kullanabileceğiniz tüm komutların listesini **docker-compose** komutunu kullanarak görebilirsiniz.
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
> Not : **docker-compose.yml** dosyası ile çalıştığınız dizinin aynı olması gerekir. Farklı dizinlerde iseniz docker engine komutları çalıştıramaz.

Bazı durumlarda compose dosyası içerisinde tüm servislerin çalışmasını istemeyebiliriz. Bunun için partial start kavramını kullanacağız. Komutumuz şöyle olacak ->  **docker-compose up service_name other_service_name**;
```
iyzi-it125:docker alican.akkus$ docker-compose up server_a server_b
Creating docker_server_a_1
Creating docker_server_b_1
Attaching to docker_server_a_1, docker_server_b_1
```
**server_c**'in çalışmasını istemedik.

Compose ile çalışan containera bağlanmayı deneyelim;
```
iyzi-it125:docker alican.akkus$ docker-compose up -d server_a server_b
Starting docker_server_a_1
Starting docker_server_b_1
iyzi-it125:docker alican.akkus$ docker-compose exec server_a /bin/bash
root@b9bb8ced17be:/# uname -a
Linux b9bb8ced17be 4.9.27-moby #1 SMP Thu May 11 04:01:18 UTC 2017 x86_64 GNU/Linux
root@b9bb8ced17be:/#
```
Compose dosyamızda verdiğimiz alias isimlendirmeler ile container birbirine erişebilir. Bunun altında yatan neden ise aynı docker engine tarafından aynı network içerisinde çalıştırılmış olmalarıdır. Örnek olarak ping atmayı deneyelim;

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

Bazı containerların çalışması için diğer bir containerın çalışmış olması gerekir. Örneğimizde web uygulamamız database ve redis cache'e bağımlıdır. Yeni bir compose dosyası oluşturalım;
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

Çıktımız şöyle olacaktır.

```
iyzi-it125:docker alican.akkus$ docker-compose up -d
WARNING: Found orphan containers (docker_server_b_1, docker_server_a_1) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
Creating docker_db_1
Creating docker_redis_1
Creating docker_app_1
iyzi-it125:docker alican.akkus$
```
Docker engine, ilk olarak bağımlı kalınan containerları çalıştırır. Ardından diğer containerlar çalışmaya başlar.

Yazımızı burada bitiriyoruz, sonraki yazılarda görüşmek üzere.

> Alican Akkus.
