---
layout: post
title: Docker containers
permalink: /blog/docker/docker-containers
summary:  Merhabalar, bu yazımızda docker container konusunu ele alacağız. Container dünyasına giriş yapmış olacağız.
---

Merhabalar, bu yazımızda docker'da container konusunu ele alacağız. Container dünyasına giriş yapmış olacağız.

### Docker containers

Image tanımını yapmaya çalışırken sık sık Container'dan yardım alınır demiştik bir önceki yazımızda. Bu iki kavram birbirinden farklı olmasına rağmen birbirini tamamlanayan/anlamlandıran yapılardır. Docker dünyasında image olmadan container, container olmadan image'in pek bir anlamı yoktur.

Container ise instance of image'dir. Image'in çalışan ve iş yapan halidir. Container oluşturmadan/run etmeden image'ın tek başına bir anlamı yoktur. Aynı image'i tekrar tekrar run edebiliriz, durumunu save edebiliriz.

Container'lar genelde tek bir sorumluluğu bulunan bir process'dir. Docker engine tarafından process'e iletilen komut'un koşturulurulmasını sağlar. Container'ların çalışma şekline bakalım;

![docker containers](/images/docker/docker-runc.png)

Docker Engine ile Container'larımız arasında **containerd** bulunur. Giriş yazımızda bu kavrama değinmiştik. Docker Engine ile container'ların koştuğu **runC** katmanı arasında köprü görevi görmektedir.

**containerd**'nin görevi **runC** process'lerini koşturmak, durdurmak ve/veya bilgi alışverişinde bulunmaktır. Asıl konumuz olan container'lar ise **runC** process'inin child process'idir. Burada **runC** ile amaç, container'ları sistem/os/platform/konum gibi değişkenlerden izole etmektir. Sonuç olarak baktığınıda elinizde bir process'den farklı bir yapı olmayacaktır.

#### Docker container vs Virtual Machines
En temel fark olarak host'daki operating system'i kullanabiliyor olmasıdır. Bu sayede hem sistem kaynaklarını kullanmada önemli ölçüde azalma meydana gelir hem de start/stop işlemleri oldukça hızlanır. Container'lar, VM'den farklı olarak donanım soyutlaması yapmaz, host'un os kernelini kullanan processcikler olarak takılırlar. Hostun birçok faydasını da alarak VM'lerde GB seviyesinde olan bir iş, Container ile MB seviyesinde yapılabilir olacaktır.

Containerlar da kendi aralarında birbirinden izole olarak çalışırlar.

Şimdi terminale geçelim ve container ile ilgili birkaç komut üzerine konuşalım;

#### List of Docker Containers
```
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
root@alican-laptop:/home/alican#
```
**docker ps** ile çalışan container'ları listeledik. Şuan çalışan(çalışmaya devam eden) herhangi bir docker container'ımız yok. Daha önce neler çalışmış ona bakalım.
**docker ps -a** ile çalışan ve **sonlanan** container'ları görelim;
```
root@alican-laptop:/home/alican# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
e12b0c69c370        docker/whalesay     "cowsay efendiler ..."   18 hours ago        Exited (0) 18 hours ago                       modest_saha
3d5d797425f6        docker/whalesay     "cowsay efendiler ..."   18 hours ago        Exited (0) 18 hours ago                       wizardly_ptolemy
a63d4fcd3ca8        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                         condescending_lovelace
c40f121af130        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                         vibrant_northcutt
682ed3e277ea        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                         stupefied_heisenberg
9d485874c155        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                         keen_williams
1f45ce04af9e        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                         eager_babbage
238a4b5cce99        9a77937b4106        "/bin/sh -c 'echo ..."   2 days ago          Exited (0) 2 days ago                         boring_kare
5befe60136df        ubuntu              "bash"                   3 days ago          Exited (130) 3 days ago                       my-ubuntu
root@alican-laptop:/home/alican#
```
Çalışıp, işi biten contaner'ları yukarda görebiliriz. Çıktı da ki alanlara bakalım;

* **CONTAINER ID** : Docker tarafından generate edilen unique id'dir. Container'ın da aynı zamanda host adıdır.
* **IMAGE** : Hangi image'ın çalışan versionu olduğunu belirtir.
* **COMMAND** : Container'a çalıştırması için iletilen komutu belirtir.
* **STATUS** : Container'ın nasıl sonlandığını ifade eder. Teorik olarak 0 dışındaki exit kodu bir hatanın oluştuğunu ifade eder. Örneğin terminalde en son **docker ps -a** komutunu çalıştırmıştık. Her process bir exit/çıkış koduna sahiptir. Terminalde **$?** ile son çalışan process'in hangi kod ile çıktını görebiliriz.
* **PORTS** : Container ile ana host arasında yada container'ın kendisinin expose ettiği port bilgilerini buradan görebiliriz.

#### Running Container
Container çalıştırmak için **docker run** komutunu kullanacağız.
```
root@alican-laptop:/home/alican# docker run ubuntu
root@alican-laptop:/home/alican#
root@alican-laptop:/home/alican# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
c393aa62ab9c        ubuntu              "/bin/bash"              8 seconds ago       Exited (0) 7 seconds ago                       wonderful_bell
e12b0c69c370        docker/whalesay     "cowsay efendiler ..."   18 hours ago        Exited (0) 18 hours ago                        modest_saha
3d5d797425f6        docker/whalesay     "cowsay efendiler ..."   18 hours ago        Exited (0) 18 hours ago                        wizardly_ptolemy
a63d4fcd3ca8        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                          condescending_lovelace
c40f121af130        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                          vibrant_northcutt
682ed3e277ea        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                          stupefied_heisenberg
9d485874c155        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                          keen_williams
1f45ce04af9e        aakkus/curl         "curl https://alic..."   2 days ago          Exited (0) 2 days ago                          eager_babbage
238a4b5cce99        9a77937b4106        "/bin/sh -c 'echo ..."   2 days ago          Exited (0) 2 days ago                          boring_kare
5befe60136df        ubuntu              "bash"                   3 days ago          Exited (130) 3 days ago                        my-ubuntu
root@alican-laptop:/home/alican#
```

**docker run ubuntu** ile ubuntu image'indan bir container oluşturduk. Dikkat ederseniz çalıştırdıktan sonra herhangi bir çıktı vs vermedi ve **docker ps** ile baktığınızda container ile alakalı bir bilgi bulamayacağız. Peki ne oldu? Container çalıştı mı? Exit kod dedik acaba hata mı aldı? **docker ps -a** ile bakalım neler olmuş.
**c393aa62ab9c** id'li container'ımız 0 exit kod ile aslında başarılı bir şekilde çalıştı ve sonlandı. Command olarak dikkat ederseniz **/bin/bash** komutunu çalıştırıyor. Dockerfile'da bunu görmeye çalışalım;
```
FROM scratch
ADD ubuntu-yakkety-core-cloudimg-amd64-root.tar.gz /

# a few minor docker-specific tweaks
# see https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap
RUN set -xe \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L40-L48
	&& echo '#!/bin/sh' > /usr/sbin/policy-rc.d \
	&& echo 'exit 101' >> /usr/sbin/policy-rc.d \
	&& chmod +x /usr/sbin/policy-rc.d \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L54-L56
	&& dpkg-divert --local --rename --add /sbin/initctl \
	&& cp -a /usr/sbin/policy-rc.d /sbin/initctl \
	&& sed -i 's/^exit.*/exit 0/' /sbin/initctl \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L71-L78
	&& echo 'force-unsafe-io' > /etc/dpkg/dpkg.cfg.d/docker-apt-speedup \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L85-L105
	&& echo 'DPkg::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' > /etc/apt/apt.conf.d/docker-clean \
	&& echo 'APT::Update::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' >> /etc/apt/apt.conf.d/docker-clean \
	&& echo 'Dir::Cache::pkgcache ""; Dir::Cache::srcpkgcache "";' >> /etc/apt/apt.conf.d/docker-clean \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L109-L115
	&& echo 'Acquire::Languages "none";' > /etc/apt/apt.conf.d/docker-no-languages \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L118-L130
	&& echo 'Acquire::GzipIndexes "true"; Acquire::CompressionTypes::Order:: "gz";' > /etc/apt/apt.conf.d/docker-gzip-indexes \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L134-L151
	&& echo 'Apt::AutoRemove::SuggestsImportant "false";' > /etc/apt/apt.conf.d/docker-autoremove-suggests

# delete all the apt list files since they're big and get stale quickly
RUN rm -rf /var/lib/apt/lists/*
# this forces "apt-get update" in dependent images, which is also good

# enable the universe
RUN sed -i 's/^#\s*\(deb.*universe\)$/\1/g' /etc/apt/sources.list

# make systemd-detect-virt return "docker"
# See: https://github.com/systemd/systemd/blob/aa0c34279ee40bce2f9681b496922dedbadfca19/src/basic/virt.c#L434
RUN mkdir -p /run/systemd && echo 'docker' > /run/systemd/container

# overwrite this with 'CMD []' in a dependent Dockerfile
CMD ["/bin/bash"]
```

**CMD ["/bin/bash"]** komutuna dikkat ediniz. Buradan çıkaracağımız ders ise, Docker Engine container'a komutu verir ve tamamlanıp tamamlanmadığını, hata alıp almadığını hem bilmez hem kontrol etmez. Aslında container'ımız kendisine verilen komutu çalıştırdı ve işini bitirerek sonlandı.

Farklı bir komut verelim ve çıktıya bakalım;
```
root@alican-laptop:/home/alican# docker run ubuntu echo Merhaba Docker kardes!
Merhaba Docker kardes!
root@alican-laptop:/home/alican#
```
```
root@alican-laptop:/home/alican# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
063740d44e57        ubuntu              "echo Merhaba Dock..."   40 seconds ago      Exited (0) 38 seconds ago                       wizardly_albattani
root@alican-laptop:/home/alican#
```
COMMAND olarak echo ile merhaba dedik. Container'ımız aldığı görevi başarıyla tamamladı ve çıktıyı terminalde gösterdi.

#### Interactive mode
Çalıştırmış olduğumuz container'a bağlanalım. Bunun için **docker run -it** Interactive mode ile başlatalım. -i interactive, -t ise attach olmayı ifade eder.
Burayı ekran görüntüsü ile ifade edelim.

![docker run](/images/docker/docker-run-1.png)

![docker run 2](/images/docker/docker-run-2.png)

Terminal de **docker run -it ubuntu** ile start edelim ve container'a root kullanıcısı ile giriş yapmış olduğumuza dikkat edelim. Aynı anda bir tab daha açıp **docker ps** ile bakalım. **docker ps -a** kullanmadığımıza dikkat edelim çünkü başlatmış olduğumuz container hala çalışır durumda.

**hostname** yazdığımızda **7b5dd** gibi bir çıktı görmekteyiz. Bu değer aslında container id değeridir. Yazıya başlarken bunu ifade etmiştik.

Şimdi bir redis image'ı çalıştıralım;
```
root@alican-laptop:/home/alican# docker run -d redis
af16d42eb9513d1bb9b6a49ef0641662c3c1bd674838fa156c93bf2bd9dbe92b
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
af16d42eb951        redis               "docker-entrypoint..."   8 seconds ago       Up 7 seconds        6379/tcp            elastic_morse
root@alican-laptop:/home/alican#
```
> docker run -d ile detached durumda başlatıyoruz container'ı. Aksi taktirde direk terminalde çalışmaya başlayacaktır.

> docker run -d redis komutundan sonra bize container id değerinin terminalde gösterildiğine dikkat edelim.

Çalışan container'a bağlanalım ve komut çalıştıralım;
```
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
af16d42eb951        redis               "docker-entrypoint..."   4 minutes ago       Up 4 minutes        6379/tcp            elastic_morse
root@alican-laptop:/home/alican# docker exec -it af16d42eb951 /bin/bash
root@af16d42eb951:/data# pwd
/data
root@af16d42eb951:/data# ls -alt
total 8
drwxr-xr-x  2 redis redis 4096 Apr 24 07:29 .
drwxr-xr-x 42 root  root  4096 Apr 24 07:29 ..
```

**docker exec -it af16d42eb951 /bin/bash** ile redis container'da bash komutunu çalıştırmak istediğimizi Docker Engine'e söyledik. Docker Engine ise containerd'ye iletti ve containerd ile runC arasında iletişim sağlanıp container'a bağlandık. Ardından standart iki komut çalıştırdık.

#### Container logs
Container loglarına şu şekilde erişebiliriz. **docker logs container-id**
```
root@alican-laptop:/home/alican# docker logs  af16d42eb951
1:C 24 Apr 07:29:59.162 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.2.8 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

1:M 24 Apr 07:29:59.164 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 24 Apr 07:29:59.164 # Server started, Redis version 3.2.8
1:M 24 Apr 07:29:59.164 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 24 Apr 07:29:59.164 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 24 Apr 07:29:59.164 * The server is now ready to accept connections on port 6379
root@alican-laptop:/home/alican#
```

#### Expose / Port Mappings
```
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
af16d42eb951        redis               "docker-entrypoint..."   4 minutes ago       Up 4 minutes        6379/tcp            elastic_morse
```
Yukarda dikkat ederseniz  PORTS kısmında 6379/tcp kısmını göreceksiniz. Redis'in Dockerfile'ına baktığınuzda ise **EXPOSE 6379** ifadesinin olduğunu görürsünüz. Container kendisini 6379 portundan dışarı açtığını söyler. Bu ifade ancak Docker network'ü içerisinde geçerlidir. Yani container to container amaçlıdır. Bu nedenle host üzerinden 6379 portuna gelecek istekler redis'e iletilmeyecektir. Telnet ile bağlanmaya çalışalım ve bağlanamadığımızı görelim;
```
root@alican-laptop:/home/alican# docker run -d redis
adf37393081858339220aee864b90252a505df0387f069788e0c393b5160f536
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
adf373930818        redis               "docker-entrypoint..."   2 seconds ago       Up 1 second         6379/tcp            tender_hermann
root@alican-laptop:/home/alican# telnet 127.0.0.1 6379
Trying 127.0.0.1...
telnet: Unable to connect to remote host: Connection refused
root@alican-laptop:/home/alican#
```
Yukarı da da ifade ettiğimiz gibi host <> container arasında bir mapping yapmadık. Yapacak şekilde deneyelim;
**docker run -d -p 6379:6379 redis** komutunu çalıştıralım, **-p 6379:6379** parametresi ile host'a 6379 portundan gelecek istekleri redisin 6379 portuna yönlendirmesini sağladık.
```
root@alican-laptop:/home/alican# docker run -d -p 6379:6379 redis
9a9aba5433b50d49f877d2ad34f8ef70a4df41156c504c6ebb6c0eec3fc41323
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
9a9aba5433b5        redis               "docker-entrypoint..."   3 seconds ago       Up 2 seconds        0.0.0.0:6379->6379/tcp   youthful_jepsen
root@alican-laptop:/home/alican# telnet 127.0.0.1 6379
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
```
PORTS kısmında öncekinden farklı olarak **0.0.0.0:6379->6379/tcp** ifadesini görmeliyiz. Öncesinde **6379/tcp** görünüyordu. Artık host üzerinden de redis'e erişebiliyoruz.
Redis cli ile ufak bir test yapalım;
```
root@alican-laptop:/home/alican/redis-3.2.8/src# ./redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set name caysever
OK
127.0.0.1:6379> get name
"caysever"
127.0.0.1:6379>
```
Başarıyla bağlandık, ping attık, birşey verdik/aldık. Her şey yolunda :)

#### Start/Stop/Pause/Restart/Remove
Container'ları hızlıca start/stop/pause/restart edebiliriz. Docker'ın hafifliği de burdan geliyor;
```
root@alican-laptop:~# docker run -d -p 6379:6379 redis
7877bbcf414a9806de909ca1c0dbee0b33bda0f6768c1d32f50a81c79b017505
root@alican-laptop:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
7877bbcf414a        redis               "docker-entrypoint..."   7 seconds ago       Up 6 seconds        0.0.0.0:6379->6379/tcp   fervent_blackwell
root@alican-laptop:~# docker pause 7877bbcf414a
7877bbcf414a
root@alican-laptop:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                    NAMES
7877bbcf414a        redis               "docker-entrypoint..."   20 seconds ago      Up 20 seconds (Paused)   0.0.0.0:6379->6379/tcp   fervent_blackwell
root@alican-laptop:~# docker unpause 7877bbcf414a
7877bbcf414a
root@alican-laptop:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
7877bbcf414a        redis               "docker-entrypoint..."   29 seconds ago      Up 28 seconds       0.0.0.0:6379->6379/tcp   fervent_blackwell
root@alican-laptop:~# docker restart 7877bbcf414a
7877bbcf414a
root@alican-laptop:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
7877bbcf414a        redis               "docker-entrypoint..."   46 seconds ago      Up 4 seconds        0.0.0.0:6379->6379/tcp   fervent_blackwell
root@alican-laptop:~# docker stop 7877bbcf414a
7877bbcf414a
root@alican-laptop:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
root@alican-laptop:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS               NAMES
7877bbcf414a        redis               "docker-entrypoint..."   2 minutes ago       Exited (0) About a minute ago                       fervent_blackwell
root@alican-laptop:~# docker rm 7877bbcf414a
7877bbcf414a
root@alican-laptop:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
> Not : Her start/stop/remove işlemlerinden sonra docker ps çıktısına dikkat ediniz.

#### Useful Container commands
Bazı durumlarda çalışan container üzerinde işlem yapmak gerekebilir;
```
root@alican-laptop:~# docker run -d -p 6379:6379 redis
ac5af1f98e0e92163de4e3c581c43e6e5482b9a1e367b84dde961e845be0de4d
root@alican-laptop:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ac5af1f98e0e        redis               "docker-entrypoint..."   2 seconds ago       Up 2 seconds        0.0.0.0:6379->6379/tcp   boring_lewin
root@alican-laptop:~# docker rename ac5af1f98e0e my-redis
root@alican-laptop:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ac5af1f98e0e        redis               "docker-entrypoint..."   23 seconds ago      Up 23 seconds       0.0.0.0:6379->6379/tcp   my-redis
root@alican-laptop:~# docker port ac5af1f98e0e
6379/tcp -> 0.0.0.0:6379
root@alican-laptop:~# docker top ac5af1f98e0e
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
999                 14419               14402               0                   11:02               ?                   00:00:00            redis-server *:6379
root@alican-laptop:~#
```
İlk olarak redis çalıştırdık. Daha sonra adını **my-redis** yaptık. Ardından  my-redis üzerindeki portları görüntüledik. Daha sonra top komutu ile process'leri listeledik. Tek process çalışmakta ve redis-server ile redisi ayağa kaldırmaktadır.

#### Save the Container state
Her **docker run** ile birlikte yeni bir container instance oluştururlur. Bu durumda geçmişte oluşturmuş olduğunuz datalar/config'ler silinebilir. Bunu engellemek için çalışan container'ın durumunu save etmek gerekir. **docker commit container-id** ile current state'i saklamış olacağız.

```
root@alican-laptop:/home/alican# docker run -d -p 6379:6379 aakkus/my-redis
5cf553e65fde81196f1ac10ca9bbb57eb829b3bc0763ce1cd8ee8517a40c016b
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
5cf553e65fde        aakkus/my-redis     "docker-entrypoint..."   16 seconds ago      Up 15 seconds       0.0.0.0:6379->6379/tcp   dazzling_hodgkin
root@alican-laptop:/home/alican# docker commit 5cf553e65fde aakkus/my-redis
sha256:0d466e451b6467ccbb62cd09da9922440f0df5742ea75f9411ca524d8d643915
root@alican-laptop:/home/alican# docker restart 5cf553e65fde
5cf553e65fde
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
5cf553e65fde        524fb1742e57        "docker-entrypoint..."   About a minute ago   Up 4 seconds        0.0.0.0:6379->6379/tcp   dazzling_hodgkin
root@alican-laptop:/home/alican#
```

Bu işlemleri yaparken diğer bir tab ile redis'e bağlıyız.
![docker redis commit](/images/docker/docker-commit-redis.png)
Son yazılan komutların çoğunun get name komutları olduğuna dikkat edin, container'ın state'ini commit edip restart ettikten sonra emin olmak için yazılan komutlar onlar :)

Yazımızı burada sonlandırıyoruz arkadaslar, bir sonraki yazımızda ise şu ana kadar yaptığımız işlere örnek olması açısından bir Spring Boot uygulamasını container üzerinde çalıştıracağız.

> Alican Akkus.
