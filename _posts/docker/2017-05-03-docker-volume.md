---
layout: post
title: Docker Volume
permalink: /blog/docker/docker-volume
summary:  Merhabalar, bu yazımızda Docker Volume ile birlikte Docker container'larda data yönetimini inceleyeceğiz.
---

Merhabalar, bu yazımızda Docker Volume ile birlikte Docker container'larda data yönetimini inceleyeceğiz.

### Docker Volume
Daha önce birçok kez Docker container'ların **runC**'in child process'i olduğunu söylemiştik. Container'ın yaşam döngüsünde start/pause/stop/remove gibi durumlar mevcut.
Container içerisinde ki datalarımızı ise persist etmemiz, gerektiğinde ise host ile paylaşmamız gerekebilir. Docker volume ile datalarımızı yönetebiliyor olacağız.

Bunun birçok faydası var şöyle ki; Container çalıştı, bir müddet iş yaptı. Birçok çıktı üretti, veri tüketti, birşeyler yazdı vs. Container'ın yaşam döngüsünden bağımsız olarak bu dataların kalıcı olması çok önemli. Misal, Bir db servis içeren container stop vs olduğunda bilgilerinizin uçmaması için önemli :)

#### Volumes Type
Docker'da data volume tutmanın/kullanmanın 3 farklı yöntemi var. İhtiyaca/yapıya göre biri seçilebilir.

1. **To keep data around when a container is removed :** Container stopped/removed durumda olsa dahi dataların silinmediği yöntem.
2. **To share data between the host filesystem and the Docker container :** Host ile container arasında dosyaların paylaşıldığı yöntem.
3. **To share data with other Docker containers :** Container'ların arasında dosyaların paylaşıldığı yöntem.

#### Keep data when a container is removed
Geçmiş yazılarda oluşturduğumuz **aakkus/movie** image'o çalıştıralım.
```
root@alican-laptop:/home/alican# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
aakkus/ubuntu                      0.1                 50166d598bef        7 days ago          174MB
aakkus/movie                       latest              bd076e582a71        7 days ago          687MB
aakkus/my-redis                    latest              0d466e451b64        9 days ago          183MB
aakkus/my-redis                    <none>              524fb1742e57        9 days ago          183MB
aakkus/curl                        latest              e9aaf778fe3e        12 days ago         186MB
microsoft/mssql-server-linux       latest              ba95cfb67ac1        2 weeks ago         1.33GB
mysql                              latest              d5127813070b        3 weeks ago         407MB
aakkus/iyzico-challenge            latest              63f7b5a24734        3 weeks ago         734MB
iyzico-challenge                   latest              63f7b5a24734        3 weeks ago         734MB
aakkus/selam-docker                latest              9a77937b4106        4 weeks ago         129MB
aakkus/pingpongwithjava            latest              ebd5d61a6b44        8 weeks ago         643MB
aakkus/caysever                    latest              677e91b6d1f4        2 months ago        129MB
nginx                              latest              6b914bbcb89e        2 months ago        182MB
arungupta/couchbase                latest              70eafed0e63c        2 months ago        583MB
redis                              latest              1a8a9ee54eb7        2 months ago        183MB
ubuntu                             latest              f49eec89601e        3 months ago        129MB
java                               8                   d23bdf5b1b1b        3 months ago        643MB
java                               latest              d23bdf5b1b1b        3 months ago        643MB
hello-world                        latest              48b5124b2768        3 months ago        1.84kB
store/hpsoftware/sitescope_store   sitescope.11.33     db14e33178d6        3 months ago        938MB
docker/whalesay                    latest              6b362a9f73eb        23 months ago       247MB
root@alican-laptop:/home/alican# docker run --name movie -d -p 8080:8080 aakkus/movie
d5c43f26a9ebef06873a3c3d027fa7573017c7585bd3e651321050d95d302d77
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d5c43f26a9eb        aakkus/movie        "java -jar /movie.jar"   3 seconds ago       Up 2 seconds        0.0.0.0:8080->8080/tcp   movie
root@alican-laptop:/home/alican#
```
Şimdi detayını görelim ve asıl üstüne konuşacağımız detaya gelelim;
```
root@alican-laptop:/home/alican# docker inspect movie
[
    {
        "Id": "d5c43f26a9ebef06873a3c3d027fa7573017c7585bd3e651321050d95d302d77",
        "Created": "2017-05-03T08:26:44.123174058Z",
        "Path": "java",
        "Args": [
            "-jar",
            "/movie.jar"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 29448,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2017-05-03T08:26:44.495972801Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:bd076e582a71b0db76fda226952d6605a0330a258a215bf9a31900718cca5b2b",
        "ResolvConfPath": "/var/lib/docker/containers/d5c43f26a9ebef06873a3c3d027fa7573017c7585bd3e651321050d95d302d77/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/d5c43f26a9ebef06873a3c3d027fa7573017c7585bd3e651321050d95d302d77/hostname",
        "HostsPath": "/var/lib/docker/containers/d5c43f26a9ebef06873a3c3d027fa7573017c7585bd3e651321050d95d302d77/hosts",
        "LogPath": "/var/lib/docker/containers/d5c43f26a9ebef06873a3c3d027fa7573017c7585bd3e651321050d95d302d77/d5c43f26a9ebef06873a3c3d027fa7573017c7585bd3e651321050d95d302d77-json.log",
        "Name": "/movie",
        "RestartCount": 0,
        "Driver": "aufs",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "8080/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": -1,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0
        },
        "GraphDriver": {
            "Data": null,
            "Name": "aufs"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "cf1616e2425b543184dc2136f549f4012ddc2909f1e0886e2151697862cff607",
                "Source": "/var/lib/docker/volumes/cf1616e2425b543184dc2136f549f4012ddc2909f1e0886e2151697862cff607/_data",
                "Destination": "/tmp",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "d5c43f26a9eb",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "8080/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64",
                "JAVA_VERSION=8u111",
                "JAVA_DEBIAN_VERSION=8u111-b14-2~bpo8+1",
                "CA_CERTIFICATES_JAVA_VERSION=20140324"
            ],
            "Cmd": null,
            "ArgsEscaped": true,
            "Image": "aakkus/movie",
            "Volumes": {
                "/tmp": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "java",
                "-jar",
                "/movie.jar"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "48c1463d2ca89c1fab7c47eacaf8a5555e482295fe7e78fedfce2c17ca4010c3",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "8080/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/48c1463d2ca8",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "520675ebb066611df2b79f4cae367c31407b4313c0265b9195c1af4704ad8996",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "4161c295afaf5b689f75cbfbe9bb326a40d5487ac17f16abfba13cd138200cb7",
                    "EndpointID": "520675ebb066611df2b79f4cae367c31407b4313c0265b9195c1af4704ad8996",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02"
                }
            }
        }
    }
]
root@alican-laptop:/home/alican#
```
Container'a ait birçok bilgiye eriştik ancak biz volume ile ilgilenelim;
```
"Mounts": [
    {
        "Type": "volume",
        "Name": "cf1616e2425b543184dc2136f549f4012ddc2909f1e0886e2151697862cff607",
        "Source": "/var/lib/docker/volumes/cf1616e2425b543184dc2136f549f4012ddc2909f1e0886e2151697862cff607/_data",
        "Destination": "/tmp",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```
Containerın run edilmesiyle birlikte mount edilen bir directory var. **Type** volume, **Destination** /tmp olan, **RW** true ile read/write olan, **Source** ile hosttaki fiziksel path'i görebiliyoruz. Burada **/tmp**'ye dikkat edelim. Çalıştırdığımız image'in Dockerfile'ına bakalım;
```
FROM java:8
VOLUME /tmp
ADD target/demo-0.0.1-SNAPSHOT.jar movie.jar
RUN bash -c 'touch /movie.jar'
EXPOSE 8080
ENTRYPOINT ["java","-jar","/movie.jar"]
```
**VOLUME /tmp** ile container'ımız kendisi için bir data volume seçmiş. Container için volume, Dockerfile'dan **VOLUME** ile belirtilebilir yada **docker run** komutu ile birlikte volume tanımı yapılabilir.

> Not: Container'ı stop edip **Source** dizinine gidip volume'ın olup olmadığına bakalım. Silinmediğini göreceğiz.

**docker run** ile volume belirleyelim;
```
root@alican-laptop:/home/alican# docker run --name movie -d -p 8080:8080 -v /aakkus aakkus/movie
8373111529b68e5720ed08470e04db75f2e6b258cdd5923cabcc8d9e3797f962
```
inspect ile bakalım;
```
root@alican-laptop:/home/alican# docker inspect movie
[
    {
        "Id": "8373111529b68e5720ed08470e04db75f2e6b258cdd5923cabcc8d9e3797f962",
        "Created": "2017-05-03T08:36:44.632447068Z",
        "Path": "java",
        "Args": [
            "-jar",
            "/movie.jar"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 30083,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2017-05-03T08:36:45.007864506Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:bd076e582a71b0db76fda226952d6605a0330a258a215bf9a31900718cca5b2b",
        "ResolvConfPath": "/var/lib/docker/containers/8373111529b68e5720ed08470e04db75f2e6b258cdd5923cabcc8d9e3797f962/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/8373111529b68e5720ed08470e04db75f2e6b258cdd5923cabcc8d9e3797f962/hostname",
        "HostsPath": "/var/lib/docker/containers/8373111529b68e5720ed08470e04db75f2e6b258cdd5923cabcc8d9e3797f962/hosts",
        "LogPath": "/var/lib/docker/containers/8373111529b68e5720ed08470e04db75f2e6b258cdd5923cabcc8d9e3797f962/8373111529b68e5720ed08470e04db75f2e6b258cdd5923cabcc8d9e3797f962-json.log",
        "Name": "/movie",
        "RestartCount": 0,
        "Driver": "aufs",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "8080/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": -1,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0
        },
        "GraphDriver": {
            "Data": null,
            "Name": "aufs"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "1dfb0e4356f0140526fe34cda6143c223ffd04c48d6e9b700dc277dc1e347d69",
                "Source": "/var/lib/docker/volumes/1dfb0e4356f0140526fe34cda6143c223ffd04c48d6e9b700dc277dc1e347d69/_data",
                "Destination": "/aakkus",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "5d572f025096ee303f4edf9a692c3b7bc65d94847d25f766fbfac171a53fa504",
                "Source": "/var/lib/docker/volumes/5d572f025096ee303f4edf9a692c3b7bc65d94847d25f766fbfac171a53fa504/_data",
                "Destination": "/tmp",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "8373111529b6",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "8080/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64",
                "JAVA_VERSION=8u111",
                "JAVA_DEBIAN_VERSION=8u111-b14-2~bpo8+1",
                "CA_CERTIFICATES_JAVA_VERSION=20140324"
            ],
            "Cmd": null,
            "ArgsEscaped": true,
            "Image": "aakkus/movie",
            "Volumes": {
                "/aakkus": {},
                "/tmp": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "java",
                "-jar",
                "/movie.jar"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "e242aa1ccbb8cf06b179f044bf4279053377bf3e764bcccc7d7183d282693199",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "8080/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/e242aa1ccbb8",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "ed9e5438140fa03cb2edc72e98b7412aa6aa1aeafe5626ce02dc59828e51e8fc",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "4161c295afaf5b689f75cbfbe9bb326a40d5487ac17f16abfba13cd138200cb7",
                    "EndpointID": "ed9e5438140fa03cb2edc72e98b7412aa6aa1aeafe5626ce02dc59828e51e8fc",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02"
                }
            }
        }
    }
]
root@alican-laptop:/home/alican#
```

Burada dikkat edecegimiz birkaç nokta var. **-v aakkus** ile volume tanımladık, alt üstü bir komut verip geçiyoruz. Asıl önemli noktaya gelelim;
```
"Volumes": {
    "/aakkus": {},
    "/tmp": {}
},
```
```
"Mounts": [
    {
        "Type": "volume",
        "Name": "1dfb0e4356f0140526fe34cda6143c223ffd04c48d6e9b700dc277dc1e347d69",
        "Source": "/var/lib/docker/volumes/1dfb0e4356f0140526fe34cda6143c223ffd04c48d6e9b700dc277dc1e347d69/_data",
        "Destination": "/aakkus",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    },
    {
        "Type": "volume",
        "Name": "5d572f025096ee303f4edf9a692c3b7bc65d94847d25f766fbfac171a53fa504",
        "Source": "/var/lib/docker/volumes/5d572f025096ee303f4edf9a692c3b7bc65d94847d25f766fbfac171a53fa504/_data",
        "Destination": "/tmp",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```
Gördüğünüz gibi **/tmp** ve **aakkus** volume'leri var görünüyor. Önemli nokta ise **aakkus** volume ile çalışmamıza rağmen **/tmp**'yi kaybetmedik.

> Not: Unutmayın, siz özellikle silmediğiniz sürece Docker datalarınızı silmez.

##### Create volume
Docker'da volume oluşturmak için **docker create** komutunu kullanacağız.
```
root@alican-laptop:/home/alican# docker volume ls
DRIVER              VOLUME NAME
root@alican-laptop:/home/alican# docker create -v /movie --name moviedata aakkus/movie
d97ad19c2e680256f92efc10c150ca73f2d5edbb1f529832757ea3dbb5397056
root@alican-laptop:/home/alican# docker volume ls
DRIVER              VOLUME NAME
local               06272346cd6df7096b22b39473ba4407f3bcc2a97239707893afae9ce8a35dd0
local               90edd95067416d0f6bb52f0e2a1f2d4fc19cfe2cceaa8c3775b8fcca423be554
root@alican-laptop:/home/alican#
```
**/movie** destination ile **moviedata** isimli bir volume oluşturduk. Oluşturduğumuz volume'ü şimdi container'a mount ederek çalıştıralım;
```
root@alican-laptop:/home/alican# docker run --volumes-from moviedata --name movie -d -p 8080:8080 aakkus/movie
5c3913b31289be4a652a0a8ea3df3843cdd2f8f40204be0b5fb2683da0c8d73c
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
5c3913b31289        aakkus/movie        "java -jar /movie.jar"   2 seconds ago       Up 1 second         0.0.0.0:8080->8080/tcp   movie
root@alican-laptop:/home/alican#
```
**--volumes-from moviedata** ile oluşturduğumuz volume'ü container'a mount ettik. Container'ı incelediğimizde **/movie** volume'ü görebiliriz;
```
"Mounts": [
            {
                "Name": "06272346cd6df7096b22b39473ba4407f3bcc2a97239707893afae9ce8a35dd0",
                "Source": "/var/lib/docker/volumes/06272346cd6df7096b22b39473ba4407f3bcc2a97239707893afae9ce8a35dd0/_data",
                "Destination": "/tmp",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Name": "90edd95067416d0f6bb52f0e2a1f2d4fc19cfe2cceaa8c3775b8fcca423be554",
                "Source": "/var/lib/docker/volumes/90edd95067416d0f6bb52f0e2a1f2d4fc19cfe2cceaa8c3775b8fcca423be554/_data",
                "Destination": "/movie",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```
Şimdi container'a girelim ve datanın persist olduğunu bizzat görelim;
```
root@alican-laptop:/home/alican# docker exec -it movie /bin/bash
root@5c3913b31289:/# ls
bin   dev  home  lib64	mnt    movie.jar  proc	run   srv  tmp	var
boot  etc  lib	 media	movie  opt	  root	sbin  sys  usr
root@5c3913b31289:/# cd movie
root@5c3913b31289:/movie# ls
root@5c3913b31289:/movie# echo "Selamun Aleykum Docker kardes." > hello
root@5c3913b31289:/movie# ls
hello
root@5c3913b31289:/movie# exit
exit
root@alican-laptop:/home/alican# docker stop 5c39
5c39
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
root@alican-laptop:/home/alican# docker start movie
movie
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
5c3913b31289        aakkus/movie        "java -jar /movie.jar"   4 minutes ago       Up 3 seconds        0.0.0.0:8080->8080/tcp   movie
root@alican-laptop:/home/alican# docker exec -it movie /bin/bash
root@5c3913b31289:/# ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  movie  movie.jar  opt  proc	root  run  sbin  srv  sys  tmp	usr  var
root@5c3913b31289:/# cd movie
root@5c3913b31289:/movie# ls
hello
root@5c3913b31289:/movie# cat hello
Selamun Aleykum Docker kardes.
root@5c3913b31289:/movie#
```
Öncelikle bash command ile giriş yaptık sonrasında movie altında bir dosya oluşturduk. Çıkıp start/stop yaptıktan sonra dosyamızı ve içeriğini bıraktığımız gibi bulduk. :)

#### Share data between the host filesystem and the Docker container
Bazı durumlarda container içerisindeki dosya/dizinlere host üzerinden erişmek isteyebiliriz. Örnek senaryomuzda nginx loglarına host üzeirinden erişmeyi gerçekleyeceğiz.
Host ile container arasında dosya paylaşım yöntemini görelim;
```
root@alican-laptop:/home/alican# cd docker/
root@alican-laptop:/home/alican/docker# mkdir nginxlogs
root@alican-laptop:/home/alican/docker# cd nginxlogs/
root@alican-laptop:/home/alican/docker/nginxlogs# pwd
/home/alican/docker/nginxlogs
root@alican-laptop:/home/alican/docker/nginxlogs# cd
root@alican-laptop:~#
```
Host filesystem'de bir dizin oluşturduk. Container'ı çalıştıralım;
```
root@alican-laptop:~# docker run -d -v /home/alican/docker/nginxlogs/:/var/log/nginx -p 5000:80 nginx
64e515c551b1a12634dba8d85c3737016e107122dcd0d513751466033ee3cb69
root@alican-laptop:~# curl localhost:5000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@alican-laptop:~#
```
**-v hostdirectory:containerdirectory** şeklinde mapping yaparak host ve container'ın ayni dizini kullanacağını ilettik.
Şimdi nginx loglarına hem hostda hem de container da bakalım;
![docker-share-directory-between-host-and-container](/images/docker/docker-share-directory-between-host-and-container.png)

Aynı log'u hem host'da hem de container üzerinden tail ettiğimizde sonucu görebiliyoruz. Resimde, yukarda yer alan terminal container'a, aşağıdaki ise host üzerindeki loglara ait. Test etmek için ise **curl localhost:5000** kullandık.

Docker bunu union file system ile birlikte copy-on-write tekniği ile halletmektedir. Bazı volume pluginler ile birlikte farklı storage'ler de container'a mount edilebiliyor.

#### To share data with other Docker containers
Container'lar arası dosya/dizin paylaşmak için ise **--volumes-from** seçeneğini kullanacağız. Başka bir container'da diğer container'a mount edilen volume'ün kendisini kullanıyor olacağız.
İki ekran görüntüsü ile bunu anlatmaya çalışalım;

![volume 1](/images/docker/volume1.png)

![volume 2](/images/docker/volume2.png)

Yaptığımız işi şöyle özetleyelim;
* **firstubuntu** adında bir container run ettik. Volume olarak **/testvolume** verdik.
* **/testvolume** altında bir dosya oluşturduk.
* **secondubuntu** isimli bir container run ediyoruz. Volume olarak **firstubuntu** ile çalışmasını söylüyoruz. Containerin içine baktığımızda kendimiz tanımlamamıza rağmen root dizinde **testvolume** adlı bir path de var. Bu dizin **firstubuntu**'dan gelen volume'dür.
* **cat** ile firstubuntu'da oluşturulan dosyanın içeriğini secondubuntu'da görebildik.

> Not: Burada dikkat edilmesi gereken konu, volüme'lerin paylaşıldığıdır. Volume directory dışındaki datalar yine container spesifiktir. Yani **testvolume** dizini dışında herhangi bir dosya paylaşılamaz.

Yazımızın sonuna gelmiş bulunmaktayız. Bir sonraki yazımızda Docker Compose'a bakacağız.

> Alican Akkus.
