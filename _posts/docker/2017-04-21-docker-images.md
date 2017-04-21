---
layout: post
title: Docker images
permalink: /blog/docker/docker-images
summary:  Merhabalar, bu yazımızda docker image konusunu ele alacağız. docker image'ın ne olduğundan, öneminden ve nasıl image oluştururuz gibi konulara değinmiş olacağız.
---

Merhabalar, bu yazımızda docker image konusunu ele alacağız. docker image'ın ne olduğundan, öneminden ve nasıl image oluştururuz gibi konulara değinmiş olacağız.

### Docker images

Image tanımını yapmaya çalışırken sık sık Container'dan yardım alınır. Image'in genel anlamı, container'lar için bir snapshot görevi görmesi ile birlikte inert(atıl/hareketsiz) ve immutable(değişmez) bir sistemi ihtiva eder. Docker'ın genel mantığını kavramak için image olayını iyice kavramak lazım.

Container ise instance of image'dir. Image'in çalışan ve iş yapan halidir. Container oluşturmadan/run etmeden image'ın tek başına bir anlamı yoktur.
Aynı image'den birden fazla container oluşturulabilir.

Image'ler docker'ın **build** komutu ile oluşturulup **run** komutu ile çalıştırılırlar. Docker CLI'daki image ile alakalı olan komutlara bakalım;

İlk olarak image'leri listeleyelim, bunun için **docker images** komutunu kullanalım;

  ![docker images](/images/docker/docker-images-new.png)

Bu komutla beraber yukardakine benzer bir çıktı aldık. Çıktıda ki alanlara bakalım;

| Field         | Value               | Description                                                     |
| ------------- |:-------------------:| ---------------------------------------------------------------:|
| REPOSITORY    | nginx/redis/ubuntu  | Build ederken -t parametresi ile image' verilen isimdir.        |
| TAG           | latest/8/sitescope  | Image'e verilen etiketir. Default olarak **latest** set edilir. |
| IMAGE ID      | ba95c../d517..      | Docker engine tarafından oluşturulan unique image id değeridir. |
| CREATED       | 2 days/ 9 days      | Image'in ne zaman build edildiğini ifade eder.                  |
| SIZE          | 1.33GB/407MB        | Image'in boyutunu ifade eder.                                   |

Image'in erişilebilir full form'u ise şöyle oluşur -> **[REGISTRYHOST/][USERNAME/]NAME[:TAG]**

Şimdi **docker info** ile genel detaylara bakalım;
  ![docker images](/images/docker/docker-info.png)

Total 23 image olduğunu, 1 Container'ın var olduğunu ve bu container'ın stop durumunda olduğunu, bir önceki yazıda değindiğimiz **runC**, **containerd** bileşenlerinin versiyonlarını, storage driver bilgilerini görebiliriz.

**docker images** ile birlikte kullanabileceğimiz komutara bakalım.

java image'lerini arayalım;
```
root@alican-laptop:/home/alican# docker images java
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
java                8                   d23bdf5b1b1b        3 months ago        643MB
java                latest              d23bdf5b1b1b        3 months ago        643MB
```

java image için digests değerlerini görüntüleyelim. Default olarak bize ilk 12 karekteri görüntüleniyordu.
```
root@alican-laptop:/home/alican# docker images java --digests
REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
java                8                   sha256:c1ff613e8ba25833d2e1940da0940c3824f03f802c449f3d1815a66b7f8c0e9d   d23bdf5b1b1b        3 months ago        643MB
java                latest              sha256:c1ff613e8ba25833d2e1940da0940c3824f03f802c449f3d1815a66b7f8c0e9d   d23bdf5b1b1b        3 months ago        643MB
```
> Dikkat ederseniz java image'ine ait iki ayrı tag ile etiketlenmiş image bulunuyor. Aynı image'ler farklı tagler ile etiketlenebilir ancak image id değerleri aynı olmalıdır.

çıktıyı formatlayalım;
```
root@alican-laptop:/home/alican# docker images --format "Name({{.Repository}})  Tag({{.Tag}})  Size({{.Size}})"
Name(microsoft/mssql-server-linux)  Tag(latest)  Size(1.33GB)
Name(mysql)  Tag(latest)  Size(407MB)
Name(aakkus/iyzico-challenge)  Tag(latest)  Size(734MB)
Name(iyzico-challenge)  Tag(latest)  Size(734MB)
Name(selam-docker)  Tag(latest)  Size(129MB)
Name(aakkus/pingpongwithjava)  Tag(latest)  Size(643MB)
Name(aakkus/caysever)  Tag(latest)  Size(129MB)
Name(nginx)  Tag(latest)  Size(182MB)
Name(arungupta/couchbase)  Tag(latest)  Size(583MB)
Name(redis)  Tag(latest)  Size(183MB)
Name(ubuntu)  Tag(latest)  Size(129MB)
Name(java)  Tag(8)  Size(643MB)
Name(java)  Tag(latest)  Size(643MB)
Name(hello-world)  Tag(latest)  Size(1.84kB)
Name(store/hpsoftware/sitescope_store)  Tag(sitescope.11.33)  Size(938MB)
Name(docker/whalesay)  Tag(latest)  Size(247MB)
```
> **--format** ile **.Id**, **.Repository**, **.Tag**, **.Size**, **.CreatedAt**, **.Digest**, **.CreatedSince** değerlerini kullanabilirsiniz.

Şimdi ise **docker image** komutlarına bakalım;

* **docker image build** :	Dockerfile'dan image build etmeyi sağlar.
* **docker image history** :	Image'a ait geçmişte yapılan işlemleri listeler.
* **docker image import** :	External image'i docker engine'e import etmeyi sağlar.
* **docker image inspect** :	Image hakkında detaylı bilgi verir.
* **docker image load** :	Var olan image'i load eder.
* **docker image ls** :	**docker images** ile aynı çıktıyı verir.
* **docker image prune** :	Kullanılmayan image'leri siler. Tehlikelidir, dikkatli kullanmak gerekir.
* **docker image pull** :	Docker hub/registry'den locale image çekmeyi sağlar.
* **docker image push** :	Docker hub/registry'e image göndermeyi sağlar.
* **docker image rm** :	Image silmeyi sağlar.
* **docker image save** :	Image'i save eder.
* **docker image tag** :	Var olan image'e yeni tag vermeyi sağlar.

Yukardaki komutların tamamına değinmeyeceğiz ama birkaçına değinelim;

**docker image history** kullanımı;
```
root@alican-laptop:/home/alican# docker image history selam-docker --human
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9a77937b4106        2 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "ec...   0B                  
55dd66b297ef        7 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "ap...   0B                  
f49eec89601e        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           3 months ago        /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
<missing>           3 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   1.9kB               
<missing>           3 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
<missing>           3 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
<missing>           3 months ago        /bin/sh -c #(nop) ADD file:68f83d996c38a09...   129MB  
```
**docker image tag** kullanımı;
```
root@alican-laptop:/home/alican# docker tag selam-docker aakkus/selam-docker
root@alican-laptop:/home/alican# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
microsoft/mssql-server-linux       latest              ba95cfb67ac1        2 days ago          1.33GB
mysql                              latest              d5127813070b        9 days ago          407MB
aakkus/iyzico-challenge            latest              63f7b5a24734        10 days ago         734MB
iyzico-challenge                   latest              63f7b5a24734        10 days ago         734MB
aakkus/selam-docker                latest              9a77937b4106        2 weeks ago         129MB
selam-docker                       latest              9a77937b4106        2 weeks ago         129MB
aakkus/pingpongwithjava            latest              ebd5d61a6b44        6 weeks ago         643MB
aakkus/caysever                    latest              677e91b6d1f4        7 weeks ago         129MB
nginx                              latest              6b914bbcb89e        7 weeks ago         182MB
arungupta/couchbase                latest              70eafed0e63c        2 months ago        583MB
redis                              latest              1a8a9ee54eb7        2 months ago        183MB
ubuntu                             latest              f49eec89601e        3 months ago        129MB
java                               8                   d23bdf5b1b1b        3 months ago        643MB
java                               latest              d23bdf5b1b1b        3 months ago        643MB
hello-world                        latest              48b5124b2768        3 months ago        1.84kB
store/hpsoftware/sitescope_store   sitescope.11.33     db14e33178d6        3 months ago        938MB
docker/whalesay                    latest              6b362a9f73eb        23 months ago       247MB
```
> selam-docker image'ını **aakkus** namespace'i altına alarak etiketledik.

**docker image rm** kullanımı;
```
root@alican-laptop:/home/alican# docker image rm selam-docker
Untagged: selam-docker:latest
root@alican-laptop:/home/alican# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
microsoft/mssql-server-linux       latest              ba95cfb67ac1        2 days ago          1.33GB
mysql                              latest              d5127813070b        9 days ago          407MB
aakkus/iyzico-challenge            latest              63f7b5a24734        10 days ago         734MB
iyzico-challenge                   latest              63f7b5a24734        10 days ago         734MB
aakkus/selam-docker                latest              9a77937b4106        2 weeks ago         129MB
aakkus/pingpongwithjava            latest              ebd5d61a6b44        6 weeks ago         643MB
aakkus/caysever                    latest              677e91b6d1f4        7 weeks ago         129MB
nginx                              latest              6b914bbcb89e        7 weeks ago         182MB
arungupta/couchbase                latest              70eafed0e63c        2 months ago        583MB
redis                              latest              1a8a9ee54eb7        2 months ago        183MB
ubuntu                             latest              f49eec89601e        3 months ago        129MB
java                               8                   d23bdf5b1b1b        3 months ago        643MB
java                               latest              d23bdf5b1b1b        3 months ago        643MB
hello-world                        latest              48b5124b2768        3 months ago        1.84kB
store/hpsoftware/sitescope_store   sitescope.11.33     db14e33178d6        3 months ago        938MB
docker/whalesay                    latest              6b362a9f73eb        23 months ago       247MB
```
> Listede artık **selam-docker** görmeyeceğiz.

**docker image push** kullanımı;
```
root@alican-laptop:/home/alican# docker image push aakkus/selam-docker
The push refers to a repository [docker.io/aakkus/selam-docker]
5eb5bd4c5014: Pushed
d195a7a18c70: Pushed
af605e724c5a: Pushed
59f161c3069d: Pushed
4f03495a4d7d: Pushed
latest: digest: sha256:bf404a5435fea43ed45e2e93436faf81607bb7cfe3048875d75c2ece919d621c size: 1357
```
> Docker [hub'a](https://hub.docker.com) giriş yapmış ve orada aynı isimde bir repository oluşturmuş olmanız gerekmektedir. **docker login** komutunu kullanarak docker cli'dan giriş yapabilirsiniz.

Hub'a push ettikten sonra görüntüleyelim;

![docker hub](/images/docker/docker-hub.png)

**docker image pull** kullanımı;
Eğer local registry içerisinde repository yok ise docker hub'dan çekilir. Eğer var ise bir işlem yapılmaz.
```
root@alican-laptop:/home/alican# docker image pull aakkus/selam-docker
Using default tag: latest
latest: Pulling from aakkus/selam-docker
Digest: sha256:bf404a5435fea43ed45e2e93436faf81607bb7cfe3048875d75c2ece919d621c
Status: Image is up to date for aakkus/selam-docker:latest
```
**docker image save** kullanımı;
```
root@alican-laptop:/home/alican# docker image save aakkus/selam-docker > ./docker/selam-docker.tar
root@alican-laptop:/home/alican# cd docker/
root@alican-laptop:/home/alican/docker# ls
docker-compose  Dockerfile  selam-docker.tar
```
**docker image inspect** kullanımı;
```
root@alican-laptop:/home/alican/docker# docker image inspect aakkus/selam-docker
[
    {
        "Id": "sha256:9a77937b4106970dddea1d3545119cac6652513d0262c1b9748260802f0faa2c",
        "RepoTags": [
            "aakkus/selam-docker:latest"
        ],
        "RepoDigests": [
            "aakkus/selam-docker@sha256:bf404a5435fea43ed45e2e93436faf81607bb7cfe3048875d75c2ece919d621c"
        ],
        "Parent": "sha256:55dd66b297efd25f5a41c7dfcbf2ad434f43fb4923309801c97f51dc952bd8b6",
        "Comment": "",
        "Created": "2017-04-02T14:13:27.545071525Z",
        "Container": "f894c4d87df77777bd55061b5d189e14912e44814d63193c1b71a02a810ed536",
        "ContainerConfig": {
            "Hostname": "e7eddde82bec",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/sh\" \"-c\" \"echo Selamun Aleykum Docker\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:55dd66b297efd25f5a41c7dfcbf2ad434f43fb4923309801c97f51dc952bd8b6",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": [],
            "Labels": {}
        },
        "DockerVersion": "17.03.1-ce",
        "Author": "",
        "Config": {
            "Hostname": "e7eddde82bec",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "echo Selamun Aleykum Docker"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:55dd66b297efd25f5a41c7dfcbf2ad434f43fb4923309801c97f51dc952bd8b6",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": [],
            "Labels": {}
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 129491393,
        "VirtualSize": 129491393,
        "GraphDriver": {
            "Data": null,
            "Name": "aufs"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:4f03495a4d7de505ccb8b8e4cfd0a8ac201491e1f67ae54b65584e0012aaab9c",
                "sha256:59f161c3069d1520deabd5823f48146232909b35573a26b9c43c63300dd814b0",
                "sha256:af605e724c5ab75671f9d21e708cff705ada13a2d1f248dd9b9d998ea35d38c8",
                "sha256:d195a7a18c704692929ebea51f235431042a294a42f2c4910b480b1e1d710cf1",
                "sha256:5eb5bd4c5014f8aec0e909341c163d4c44baf750e31691e7c8fdc2beb6c55de9"
            ]
        }
    }
]
```

Şimdi ise Dockerfile yardımı ile bir image oluşturup devamında çalıştıralım;

**Dockerfile**'ın özel bir dosya olduğunu ifade etmiştik. XML/JSON/YML gibi belirli bir yapısı yoktur. Satır satır instruction'lardan oluşur. Her instruction bir step özelliği taşır ve Docker engine tarafından koşturulur.

```
#extends from latest ubuntu image
from ubuntu:latest

#maintainer
MAINTAINER aakkus <alican.akkus94@gmail.com>

#update packages
run apt-get -y update

#install curl
run apt-get -y install curl

#entry point curl
ENTRYPOINT [ "curl" ]
```
> Not : Bir önceki yazıda Dockerfile içerisinde kullanılabilecek Instruction’lardan bahsetmiştik. Yukarıdaki örnek dosyamızda birkaç'ını kullanmış olduk.

Image'imiz ubuntu'dan miras alıp üzerine curl kurma işlemini yapıyor. Çalıştırma anında verilecek parametre için curl çalışacaktır.
CMD'den farklı olarak ENTRYPOINT kullanımı daha çok docker run komutu kullanılırken parametre verilecek image'ler için kullanılması uygundur.

> Not : ENTRYPOINT ["curl"] komutunun CMD olarak karşılığı ise şöyledir ->  CMD [ "curl", "https://alicanakkus.github.io" ]
Farklı olarak curl ile alacağımız adresi parametre olarak değil file'da belirtmiş olduk.

Build edelim;
```
root@alican-laptop:/home/alican/docker/docker-curl# docker build -t aakkus/curl .
Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM ubuntu:latest
 ---> f49eec89601e
Step 2/5 : MAINTAINER aakkus <alican.akkus94@gmail.com>
 ---> Using cache
 ---> a69daac4b3ce
Step 3/5 : RUN apt-get -y update
 ---> Using cache
 ---> 8ece943434fb
Step 4/5 : RUN apt-get -y install curl
 ---> Using cache
 ---> 0442ddbb0c61
Step 5/5 : ENTRYPOINT curl
 ---> Using cache
 ---> e9aaf778fe3e
Successfully built e9aaf778fe3e
root@alican-laptop:/home/alican/docker/docker-curl#
```
Dockerfile'ın olduğunu dizine gelip **docker build -t aakkus/curl .** komutu ile build ediyoruz. Yukarıdaki çıktıda apt-get kısımlarında **Using cache** ifadesi bulunuyor. Image ilk kez build edildiğinde paketlerin indirildiğini göreceksiniz. Daha sonraki buildlerde ise Docker akıllı davranarak cache'den bularak işlem yaptı.

> Not : Build komutu içerisinde ki **.** ifadesi current directory'i işaret eder. Bu da Dockerfile'ın nerede olduğunu belirtir. Eğer çalışılan dizin ile Dockerfile'ın bulunduğu dizin farklı ise **.** yerine Dockerfile'ın path'i verilmelidir.

Container oluşturalım;
```
root@alican-laptop:/home/alican/docker/docker-curl# docker run aakkus/curl https://alicanakkus.github.io > curl-blog.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 20081  100 20081    0     0  27928      0 --:--:-- --:--:-- --:--:-- 27929
root@alican-laptop:/home/alican/docker/docker-curl# head -15 curl-blog.txt
<!doctype html>
<!--[if lt IE 7]><html class="no-js lt-ie9 lt-ie8 lt-ie7" lang="en"> <![endif]-->
<!--[if (IE 7)&!(IEMobile)]><html class="no-js lt-ie9 lt-ie8" lang="en"><![endif]-->
<!--[if (IE 8)&!(IEMobile)]><html class="no-js lt-ie9" lang="en"><![endif]-->
<!--[if gt IE 8]><!--> <html class="no-js" lang="en"><!--<![endif]-->
<head>
<meta charset="utf-8">
<title>Taze Yazılar &#8211; Çaysever - JavaMan</title>




<!-- Twitter Cards -->
<meta name="twitter:title" content="Taze Yazılar">

root@alican-laptop:/home/alican/docker/docker-curl#
```

**docker run aakkus/curl https://alicanakkus.github.io > curl-blog.txt** olan komutumuz ile **https://alicanakkus.github.io** adresini çekmesini istedik ve curl-blog.txt dosyasına içeriği yazmasını istedik.

#### Docker images best practice

* **.dockerignore** dosyası kullanmak. Host'dan image'e kopyalanacak olan dizinlerde vs kopyalanmaması istenilen dosyaları buraya ekleyebiliriz. **.gitignore** gibi düşünebilirsiniz.
* Gereksiz paketlerin download/install edilmeisini engellemek.
* Her image'in çalışan örneği olan container'ın tek bir sorumluluğu olması lazım. Bakımı kolay, genişletilebilir, ihtiyaca göre küçülebilen/büyüyebilen sistemler için her container'ın tek bir sorumluluğu olması gerekir.
* Development ortamında production ortamına kadar ki süreç içerisinde aynı image üzerinde işlem yapılması gerekir.
* Continuous integration and continuous delivery pipeline'ları için farklı registry/hub kullanılması önerilir.

Yazıyı burada sonlandırıyoruz, bir sonraki yazıda Docker Container kavramına değineceğiz.

> Alican Akkus.
