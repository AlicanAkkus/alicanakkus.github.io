---
layout: post
title: Docker Network
permalink: /blog/docker/docker-network
summary:  Merhabalar, bu yazımızda Docker Network konusunu işleyeceğiz.
---

Merhabalar, bu yazımızda Docker Network konusunu işleyeceğiz.

### Docker Network
Docker'ın çıkış amaçlarından biri de sistemlerin birbirinden izole olmasının sağlanması, fail durumlarında sadece kendisini etkilemesi, sistemler/mimariler/uygulamalar arasında loose coupling bir bağın olmasının sağlanmasıdır.

Teorik olarak container'lar birbirinden izole/insensible durumdadır. Ancak container'ları birbirine bağlayabiliriz, aynı network içerisindeymiş gibi birbirleri ile iletişime geçmelerini sağlayabiliriz.

Bugün Docker network konusunu inceleyerek yukarda ifade ettiğim durumları gerçekleştireceğiz.

Öncelikle bir docker installation'a sahip iseniz default olarak bazı network katmanları otomatik olarak oluşturulur. Extra herhangi bir işlem yapmanıza gerek yoktur.
**docker network ls** ile installation ile birlikte gelen network'lere bakalım;
```
root@alican-laptop:/home/alican# docker network ls
NETWORK ID          NAME                    DRIVER              SCOPE
e86c8dff122b        bridge                  bridge              local
fbeffb7d7932        dockercompose_default   bridge              local
ce57bad8736c        host                    host                local
e8ac14162720        none                    null                local
root@alican-laptop:/home/alican#
```
Bende docker compose da bulunduğu için **dockercompose_default** network'ü de görmekteyiz. Diğerleri docker'a ait. Host makinamızda **ifconfig** çalıştıralım;
```
root@alican-laptop:/home/alican# ifconfig
br-fbeffb7d7932 Link encap:Ethernet  HWaddr 02:42:93:26:81:91  
          inet addr:172.18.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

docker0   Link encap:Ethernet  HWaddr 02:42:eb:48:a1:73  
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:ebff:fe48:a173/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:46231 errors:0 dropped:0 overruns:0 frame:0
          TX packets:86074 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:2444072 (2.4 MB)  TX bytes:164240226 (164.2 MB)
```
**br-fbeffb7d7932** interface name'li network docker-compose'a ait, **docker0** ise docker'ın **bridge** network'e ait. **bridge** özel bir network docker için. Onun detayına bakalım;

#### Docker **bridge** network
**bridge** network, docker installation ile birlikte gelen ve her yeni bir container'ın oluşturulması ile default olarak dahil olduğu bir network'dur. Yani siz herhangi bir container çalıştırdığınızda özellikle belirtmedi iseniz **bridge** network'üne dahil olacaktır container. Burdan şu sonucu çıkarabiliriz;
> Default olarak containerlar, **bridge** network'e dahil oldukları için aslında birbirileri ile de iletişime geçebilirler.

Bridge network'e detaylıca bakalım;
```
root@alican-laptop:/home/alican# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "e86c8dff122b17312d7696e3af8c3a282a15aca5af424a484d180f815e25e1df",
        "Created": "2017-04-26T07:49:17.481732288+03:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
root@alican-laptop:/home/alican#
```
**docker network inspect bridge** komutu ile **bridge** network'ün detaylarını görebiliyoruz. Adı, ne zaman oluşturulduğu, options'ları, bağlı container'ları vs görebiliriz.
Root'un altındaki **Containers** objesinin altında network'e bağlanmış container'ları göreceğiz. Çalışan container'ımız bulunmadığı için şuan içini boş görüyoruz.

Şimdi iki adet ubuntu container oluşturalım ve tekrar bakalım;

**ubuntu1**
```
root@alican-laptop:/home/alican# docker run -it --name=ubuntu1 aakkus/ubuntu:0.1
root@48db53eb2d40:/#
```
**ubuntu2**
```
root@alican-laptop:/home/alican# docker run -it --name=ubuntu2 aakkus/ubuntu:0.1
root@de6203ce3784:/#
```
Çalışan container'ları görelim;
```
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
de6203ce3784        aakkus/ubuntu:0.1   "/bin/bash"         About a minute ago   Up About a minute                       ubuntu2
48db53eb2d40        aakkus/ubuntu:0.1   "/bin/bash"         2 minutes ago        Up 2 minutes                            ubuntu1
root@alican-laptop:/home/alican#
```

> Not: aakkus/ubuntu image'ı ubuntu:latest image'den miras alır ve bazı network toollarını içerir. Docker hub'dan **docker pull aakkus/ubuntu** şeklinde local'e çekebilirsiniz.

İki adet ubuntu container'ımız çalışıyor. Şimdi bridge inspect yapalım;
```
root@alican-laptop:/home/alican# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "e86c8dff122b17312d7696e3af8c3a282a15aca5af424a484d180f815e25e1df",
        "Created": "2017-04-26T07:49:17.481732288+03:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "48db53eb2d404556c3d384cb0152ae67f079dbe432cc9d0873f893ebf826c17e": {
                "Name": "ubuntu1",
                "EndpointID": "da13163eb668607cda457a7e9b043badbd8d613ce9b704d156b0fa572877cdc9",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "de6203ce3784ca1da35c600eb7ac241678abb71a6ff29a0beeec0aecb60bcd02": {
                "Name": "ubuntu2",
                "EndpointID": "35f26c2d81b984cf95368a7794fb123178f23cb58b2ea67d37380ec4f3a889b6",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
root@alican-laptop:/home/alican#
```
**docker network inspect bridge** komutunu ilk çalıştırdığımız halinden farklı olarak **Containers** kısmında iki adet container olduğunu görüyoruz. Bu container'lara ait **name**, **ip** gibi bilgiler mevcut.

* **ubuntu1** container'a 172.17.0.2 ip'si, **ubuntu2**'ye ise 172.17.0.3 ip adresi atanmış.

**ubuntu1** container'ın içerisinden **ubuntu2** container'a ping atalım;
```
root@48db53eb2d40:/# hostname
48db53eb2d40
root@48db53eb2d40:/# ping -w3 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.179 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.097 ms
64 bytes from 172.17.0.3: icmp_seq=3 ttl=64 time=0.130 ms
64 bytes from 172.17.0.3: icmp_seq=4 ttl=64 time=0.115 ms

--- 172.17.0.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2998ms
rtt min/avg/max/mdev = 0.097/0.130/0.179/0.031 ms
root@48db53eb2d40:/#
```
Tersini yapalım;
```
root@de6203ce3784:/# hostname
de6203ce3784
root@de6203ce3784:/# ping -w3 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.132 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.093 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.091 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.094 ms

--- 172.17.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2997ms
rtt min/avg/max/mdev = 0.091/0.102/0.132/0.019 ms
root@de6203ce3784:/#
```
İki container birbirine erişebilir durumda. **ubuntu2** contaier'ını bridge network'den atalım ve tekrar deneyelim aynı işlemi;
```
root@alican-laptop:/home/alican# docker network disconnect bridge ubuntu2
root@alican-laptop:/home/alican# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "e86c8dff122b17312d7696e3af8c3a282a15aca5af424a484d180f815e25e1df",
        "Created": "2017-04-26T07:49:17.481732288+03:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "48db53eb2d404556c3d384cb0152ae67f079dbe432cc9d0873f893ebf826c17e": {
                "Name": "ubuntu1",
                "EndpointID": "da13163eb668607cda457a7e9b043badbd8d613ce9b704d156b0fa572877cdc9",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
root@alican-laptop:/home/alican#
```
**docker network disconnect networkName containerName** komutu ile ubuntu2'yi bridge'ın dışına aldık. Tekrar deneyelim ve ulaşamadığımızı görelim;
```
root@48db53eb2d40:/# ping -w3 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.179 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.097 ms
64 bytes from 172.17.0.3: icmp_seq=3 ttl=64 time=0.130 ms
64 bytes from 172.17.0.3: icmp_seq=4 ttl=64 time=0.115 ms

--- 172.17.0.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2998ms
rtt min/avg/max/mdev = 0.097/0.130/0.179/0.031 ms
root@48db53eb2d40:/#
root@48db53eb2d40:/#
root@48db53eb2d40:/# ping -w3 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.

--- 172.17.0.3 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2016ms

root@48db53eb2d40:/#
```
Disconnect ettikten sonra **ubuntu2** container'ı networksüz kaldı. Tekrar connect olalım;
```
root@alican-laptop:/home/alican# docker network connect bridge ubuntu2
root@alican-laptop:/home/alican# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "e86c8dff122b17312d7696e3af8c3a282a15aca5af424a484d180f815e25e1df",
        "Created": "2017-04-26T07:49:17.481732288+03:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "48db53eb2d404556c3d384cb0152ae67f079dbe432cc9d0873f893ebf826c17e": {
                "Name": "ubuntu1",
                "EndpointID": "da13163eb668607cda457a7e9b043badbd8d613ce9b704d156b0fa572877cdc9",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "de6203ce3784ca1da35c600eb7ac241678abb71a6ff29a0beeec0aecb60bcd02": {
                "Name": "ubuntu2",
                "EndpointID": "eb247453ea55199b21ccbf4f83c001c5280b3ddb921d9edb2a5a5b435075da79",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
root@alican-laptop:/home/alican#
```

#### Docker **host** network
Bridge'den farklı olarak host network ile çalıştırılırsa container, host'daki tüm network interface'e erişebilir olacaktır. **host** modda çalışacak **ubuntu3** container'ı oluşturalım.

```
root@alican-laptop:/home/alican# docker run -it --network=host --name=ubuntu3 aakkus/ubuntu:0.1
```

> Bir container'ı farklı network'de çalıştırmak isterseniz **--network** parametresi ile network interface adını vermeniz yeterlidir.

**ubuntu3** deki network interface'lere bakalım;
```
root@alican-laptop:/# ifconfig
br-fbeffb7d7932 Link encap:Ethernet  HWaddr 02:42:93:26:81:91  
          inet addr:172.18.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

docker0   Link encap:Ethernet  HWaddr 02:42:eb:48:a1:73  
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:46232 errors:0 dropped:0 overruns:0 frame:0
          TX packets:86075 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:2444100 (2.4 MB)  TX bytes:164240413 (164.2 MB)

eth0      Link encap:Ethernet  HWaddr 68:f7:28:46:39:a5  
          inet addr:XXXXXXXX  Bcast:10.255.255.255  Mask:255.0.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:5614471 errors:0 dropped:0 overruns:0 frame:0
          TX packets:5066202 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2249640167 (2.2 GB)  TX bytes:4801685943 (4.8 GB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:319525 errors:0 dropped:0 overruns:0 frame:0
          TX packets:319525 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:64779805 (64.7 MB)  TX bytes:64779805 (64.7 MB)

veth5b89fb9 Link encap:Ethernet  HWaddr b2:4e:c4:5a:ef:cb  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:28 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:4543 (4.5 KB)

veth86867c7 Link encap:Ethernet  HWaddr ca:d4:4a:ca:76:62  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:16 errors:0 dropped:0 overruns:0 frame:0
          TX packets:40 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1288 (1.2 KB)  TX bytes:5550 (5.5 KB)

vmnet1    Link encap:Ethernet  HWaddr 00:50:56:c0:00:01  
          inet addr:192.168.51.1  Bcast:192.168.51.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2015 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

vmnet8    Link encap:Ethernet  HWaddr 00:50:56:c0:00:08  
          inet addr:172.16.85.1  Bcast:172.16.85.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2015 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

wlan0     Link encap:Ethernet  HWaddr 48:51:b7:a0:73:55  
          inet addr:192.168.1.126  Bcast:192.168.1.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:21589 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9416 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2053323 (2.0 MB)  TX bytes:962260 (962.2 KB)

root@alican-laptop:/#
```
Host'daki network'ün kopyası **ubuntu3** için de tanımlı oldu. Host network'i inspect edelim;
```
root@alican-laptop:/home/alican# docker network inspect host
[
    {
        "Name": "host",
        "Id": "ce57bad8736c9224843bed9352e5978daf567095cda8685a24b9e0792e081af3",
        "Created": "2017-02-27T12:42:32.808387833+03:00",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "018869cdbe14e80f1be936d893aafb5fb4fbdfb27a6a8549e18df0867ec2a55c": {
                "Name": "ubuntu3",
                "EndpointID": "9b19e7fc38034927a297c5cfaa2b902f1f3fcccc96f7ddb164a957e67446499f",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
root@alican-laptop:/home/alican#
```
#### Docker **none** network
Bir container'ın herhangi bir network'e sahip olmadığı durumdur. None olarak çalıştırılan container'lar docker network stack'ine alınırlar ancak herhangi bir network configuration'u yapılmaz. Şöyle ki;
```
root@alican-laptop:/home/alican# docker run -it --network=none --name=ubuntu4 aakkus/ubuntu:0.1
root@bccd5cdb5348:/# ifconfig
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

root@bccd5cdb5348:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
root@bccd5cdb5348:/#
```
**ubuntu4** container **none** modunda çalıştırıldı. Ve sadece **lo** interface tanımlı, o da iç network'de yer alan herhangi bir ip adresine erişmek istenirse dış network'e bulaşmaması için loopback interface'dir.

#### Docker Service Discovery
Docker container'lar aynı network içerisinde iseler ip adresi üzerinden birbirine erişebilirler. Container name ile resolve edebilmesi için docker'a container'ları birbirinle link'leme yapmasını söylemelisiniz. Ancak bu yöntem pek önerilmez.
```
root@alican-laptop:/home/alican# docker run -it --link=ubuntu1 --network=bridge --name=ubuntu5 aakkus/ubuntu:0.1
root@3815ebc8ce76:/# ping -w3 ubuntu1
PING ubuntu1 (172.17.0.2) 56(84) bytes of data.
64 bytes from ubuntu1 (172.17.0.2): icmp_seq=1 ttl=64 time=0.182 ms
64 bytes from ubuntu1 (172.17.0.2): icmp_seq=2 ttl=64 time=0.046 ms
64 bytes from ubuntu1 (172.17.0.2): icmp_seq=3 ttl=64 time=0.046 ms
64 bytes from ubuntu1 (172.17.0.2): icmp_seq=4 ttl=64 time=0.094 ms

--- ubuntu1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2997ms
rtt min/avg/max/mdev = 0.046/0.092/0.182/0.055 ms
root@3815ebc8ce76:/#
```
**--link** parametresi ile container'ları birbirine bağlayabilir ve container name ile resolve etmesini sağlamış olursunuz.

#### Docker user-defined networks
Bazı durumlarda kendi network'ünüzü oluşturup, sistemleri/uygulamaları gruplayabilir/sınırlandırabilirsiniz. Kendi network'ümüzü oluşturalım;
```
root@alican-laptop:/home/alican# docker network create --driver bridge caysever
9216aa953a7340a5449d6fe6595ada8a388da6306ff3a1ec94bc4fcd0efbf46a
root@alican-laptop:/home/alican# docker network ls
NETWORK ID          NAME                    DRIVER              SCOPE
e86c8dff122b        bridge                  bridge              local
9216aa953a73        caysever                bridge              local
fbeffb7d7932        dockercompose_default   bridge              local
ce57bad8736c        host                    host                local
e8ac14162720        none                    null                local
root@alican-laptop:/home/alican# docker network inspect caysever
[
    {
        "Name": "caysever",
        "Id": "9216aa953a7340a5449d6fe6595ada8a388da6306ff3a1ec94bc4fcd0efbf46a",
        "Created": "2017-04-27T11:09:03.118268926+03:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
root@alican-laptop:/home/alican#
```
**docker network create --driver bridge caysever** komutu ile **caysever** adında network oluşturduk. caysever network'de container oluşturmak için ise;
```
root@alican-laptop:/home/alican# docker run -it --network=caysever --name=ubuntu6 aakkus/ubuntu:0.1
root@e7e87b4aa17b:/#
root@alican-laptop:/home/alican# docker network inspect caysever
[
    {
        "Name": "caysever",
        "Id": "9216aa953a7340a5449d6fe6595ada8a388da6306ff3a1ec94bc4fcd0efbf46a",
        "Created": "2017-04-27T11:09:03.118268926+03:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "e7e87b4aa17b9b60088e599f3427ec1c84dd3fa11dfdaa37df2e4da82427623d": {
                "Name": "ubuntu6",
                "EndpointID": "5a790ffe80bfca6ea7e957139c2a1990afbf16c82bd280009c322a2669f4dd1b",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
root@alican-laptop:/home/alican#
```

Son olarak çalıştırdığımız container'ları silelim;
```
root@alican-laptop:/home/alican# docker rm $(docker ps -a -q)
d129e2a3fdc2
78f367097772
018869cdbe14
Error response from daemon: You cannot remove a running container e7e87b4aa17b9b60088e599f3427ec1c84dd3fa11dfdaa37df2e4da82427623d. Stop the container before attempting removal or force remove
Error response from daemon: You cannot remove a running container de6203ce3784ca1da35c600eb7ac241678abb71a6ff29a0beeec0aecb60bcd02. Stop the container before attempting removal or force remove
Error response from daemon: You cannot remove a running container 48db53eb2d404556c3d384cb0152ae67f079dbe432cc9d0873f893ebf826c17e. Stop the container before attempting removal or force remove
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e7e87b4aa17b        aakkus/ubuntu:0.1   "/bin/bash"         12 minutes ago      Up 12 minutes                           ubuntu6
de6203ce3784        aakkus/ubuntu:0.1   "/bin/bash"         About an hour ago   Up About an hour                        ubuntu2
48db53eb2d40        aakkus/ubuntu:0.1   "/bin/bash"         About an hour ago   Up About an hour                        ubuntu1
root@alican-laptop:/home/alican# docker rm $(docker ps -a -q) -f
e7e87b4aa17b
de6203ce3784
48db53eb2d40
root@alican-laptop:/home/alican#
```
**docker rm $(docker ps -a -q)** ile çalışan tüm container'ları silebilirsiniz. Docker çalışan container'ları silmenize izin vermez ancak **-f** parametresi ile force edebilirsiniz, pek önerilmez :)

Yazımızı burada sonlandırıyoruz, bir sonraki yazımızda Docker Volume konusuna değineceğiz.

> Alican Akkus.
