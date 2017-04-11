---
layout: post
title: Docker Introduction
permalink: /blog/docker/docker-introduction
summary: Merhabalar, Docker teknolojisine giriş yapmış olacağız. Docker'dan ve çözmeyi vadettiği konuları ele alacağız.
---

Merhabalar, Docker teknolojisine giriş yapmış olacağız. Docker'dan ve çözmeyi vadettiği konuları ele alacağız.

### Docker

Docker, temelde bir sanallaştırma teknolojisi olarak tarif edilebilir. Temelinde ise Linux Containers vardır. Linux Container(LXC) üzerine kurulmuş birbirinden izole olarak çalışabilen Container'lardan oluşan bir yapıdır. Uygulama/process sanallaştırma işlemini yapar. Yani uygulama bağımlılıklarını minimize eder ve bir iki işlem ile uygulamaların/process'lerin koşturulmasını sağlar. Kısacası Docker ile bir uygulamanın dağıtılması, paylaşılması kolaylaştırılmaktadır.

> LXC, 2008 yılında Linux Kernel'e eklenen bir yapıdır.

Docker'ın emekleme dönemeinde var olan LXC daha sonra yerini **runC** bıraktı. Docker tarafından alınan önemli bir karardı. Çünkü kendi topluluklarını yaratmalarını sağladı hemde Linux'dan bağımsız olarak container kavramında genişletmeler/geliştirmeler yapabildiler. Daha sonra [containerd](https://containerd.io) ile bir standart getirdiler. Bununla birlikte LXC de yerini **runC**'ye bırakmış oldu.

VMware, Hyper-V gibi sanallaştırma mimarilerinden farklı olarak işletim sistemi seviyesinde bir sanallaştırma yaptığından dolayı fiziksel sunucularda yapılan sanallaştırmalara oranla daha az kaynak ayırarak kaynakları daha efektik kullanmayı sağlar.

Ayrıca Hypervisor sanallaştırma da farklı operating systemler olduğundan dolayı host ile birlikte toplam 2 tane işletim sisteminin bakımı, kontrolü ve yönetilmesi gerekmektedir. Docker ise LXC ile birlikte host'un işletim sistemi üzerinde koştuğu için bu yönden de bir avantajı vardır.

Docker ile birlikte development ortamlarının kurulumu olsun, uygulamaların taşınabilirliği olsun birçok konuda artık elimiz rahatlayacaktır. Örneğin müşterinize docker image hazırlayıp sadece run etmesini söyleyebilirsiniz. Yada şirkette yeni bir arkadaş işe girdi, kendisine çalışma ortamı hazırlanacak, docker ile 15 dk da artık çalışabilir duruma gelebilir :) Bu sayede bir geliştirme ortamın da bir standartlaşma oluşuacaktır.

Docker'ın bir diğer faydası ise "benim makinam da çalışıyordu ya" problemini çözmesidir.

Mikroservis konusu ise başlı başına bir konu. Docker Mikroservis mimarisi için biçilmez bir kaftan. Artan ihtiyaçlar karşısında yeni bir node ayağa kaldırmak oldukçça kolaylaşacaktır.

Docker kurulumu için [şuradan](https://docs.docker.com/engine/installation/) yardım alabilirsiniz. Docker'ın iki versionu var görünmekte. Docker EE, enterprise işleri için kullanılan ücretli bir sürümdür. Docker CE, community edition olarak sulunuyor. Bunu kurabilirsiniz. Docker kurduktan sonra docker versionu şöyle görebilirsiniz;

![docker version](/images/docker/docker-version.png)

Docker konusunda bazı keywordler mevcut bunlara da detaylıca bakalım;
* Docker Engine : Docker daemon olarak da karşımıza çıkabilir. Sanallaştırmanın yapıldığı kısımdır. VMware, Hyper-V yapılarına karşılık gelmektedir.
* runC : Docker'ın LXC'den sonra geçiş yaptığı yeni sanallaştırma aracıdır.
* Docker CLI : Docker engine üzerinde komutların koşturulmasını sağlayan istemcidir.
* DockerFile : Bir docker uygulaması oluşturabilmek için gerekli dosyadır. Bu dosya içerisinde uygulamanın neler yaptığını görebiliriz.
* Images : Bir imaj dosyasıdır. DockerFile'ın build edilmesi ile oluşur.
* Container : Bir image'in çalışan örneğidir. Image'in çalışan versionu container'dır diyebiliriz.
* Docker Registry/Hub : Docker registry/hub sayesinde farklı amaçlar için üretilmiş image'ler incelenebilir, kullanılabilirdir. Maven repository'e benzetilebilir. Buraya kendi oluşturduğumuz image'leri de gönderebiliriz.

#### runC

Docker Engine altında ki **containerd** aracı docker engine ile container'lar arasında ki iletişimden sorumludur.

![docker runC](/images/docker/docker-runc.png)

Image'ın çalışan örneği olan container'lar ise runC processinin child processi olarak çalışır.

#### Docker CLI

Docker CLI(Command Line Interface) ile docker komutlarını koşturabilieceğiniz bir arayüzdür. Docker CLI TCP üzerinden Docker Daemon(Docker Engine) ile konuşarak komutları kendisine iletir ve işletilen komutların çıktısını gösterir. TCP/IP üzerinden bağlantı olduğu için uzak sucunudaki Docker daemon'a Docker CLI ile bağlanıp komutlar da koşturulabilir.

#### DockerFile

DockerFile özel bir dosyadır. Bu dosya içerisinde image'a ait özellikler bulunur. Bu dosya aracılığı ile bir docker image dosyası oluşturabiliriz. DockerFile içerisinde kullanabileceğimiz özel keywordler mevcut. Bu keywordler ile image build etme konularına da değineceğiz. Bu keywordler'den bazıları şöyle, daha sonra detaylı olarak DockerFile oluştururken değineceğiz;

* FROM : Var olan bir image'den extends etme. Miras alma gibi.
* COPY : Host'dan docker container'a dosya kopyalamak için kullanılabilir.
* RUN : Container'da bir komut çalıştırmak içindir.
* ENV : Container üzerinde kullanabilmek için environment variable set etmek içindir.
* WORKDIR : Working Directory'dir, çalışılan dizini değiştirmeye yarar.
* MAINTAINER : Image'in kim tarafından oluşturduğunu belirtir. Image'i kullanan birisi maintainer'a bakarak feedback gönderebiliri. Kullanılması önerilir.
* EXPOSE : Container'ı dışarıya verilen porttan açabilmeyi sağlar.

#### IMAGES

Image'ler bir kontrattır yada tanımdır diyebiliriz, çalışan bir sistem değildir. Herhangi bir image'in yapısına bakarak ne iş yaptığını az çok tahmin edebiliriz. Docker image'ı başka bir image'den türeyebilir. Bir Docker image'ı aynı anda birdan fazla sorumluluğa sahip olabilir. Ancak bu önerilen bir şey değildir. Örneğin, bir docker image'ı aynı anda app server, web server, db işlerini görebilir. Ancak bakımı zor ve esnek olmayan bir yapı olmuş olur. Yarın baktınız ki web server istekleri işlemekte zorlanıyor ve dediniz ki bu image'den çalışan bir örnek(container) daha oluşturalım. Bu durumda web server ihtiyacından dolayı gereksiz yere app server ve db'yi de genişletmiş oldunuz.

Bu nedenle her docker image'ın tek bir sorumluluğu olmalı. Bu sayede bakımı kolay, scale edilebilen, ihtiyaca göre büyüyüp küçülebilen sistemler oluşturulabilir.

Docker CLI ile image'leri bir görelim;

Terminalden **docker images** komutunu koşturalım ve sonuca bakalım;
![docker images](/images/docker/docker-images.png)

Docker CLI ile Docker Engine konuşarak makine üzerinde kullanılabilecek olan image'leri listeledi. Her bir image onu tanımlayan birkaç metadata'ya sahiptir. Bunlara bakalım;

* repository : docker image'in adıdır.
* tag : versionlama için faydalıdır. repository adı ile birlikte unique bir identifier oluşturur.
* image id : Docker tarafındna image için oluşturulan id değeridir.
* created ve size : bla bla :)

İmage id Docker tarafından üretilen ve SHA'sı alınmış bir değerdir. Yukarıdaki komut ile ilk 12 karekteri görüntülenir. Hepsini görmek için şu komutu girelim **docker images --no-trunc -q**;

![docker images id](/images/docker/docker-images-id.png)

#### CONTAINERS

Container'ların image'lerin çalışan bir örneği olduğunu ifade ettik. İmage'ler tek başına bir anlam ifade etmezler. Her docker image'ı docker engine ile başlatılır ve artık container olmuş ve çalışan bir sistem ihtiva eder.

Bir image çalıştırıp sonucuna bakalım;
![docker run](/images/docker/docker-run.png)

**docker run aakkus/caysever** ile docker engine **aakkus/caysever** imajından bir çalışan örnek/instance oluşturmasını istedik. Çıktı olarak da Hello Dockerrrr gibi bir çıktı verdi. Docker image'ı burada ekrana hello yazdırmak için hazırlanmış. Docker temelde DockerFile da yer alan komutu container'a vererek kendi işini bitirir. Container o komut ile ne yapar, ne zaman biter onla ilgilenmez. Yukarıdaki image'ın DockerFile'ına bakalım;

```
from ubuntu

cmd apt-get -y update

cmd echo Hello Dockerrrrr
```

İmajımız ubuntu imajından türetilmiş olup iki adet komut çalıştırarak işini sonlandırıyor.

Çalıştırdığımız container'ı bir sorgulayalım. **docker ps** ile çalışan instance'ları görelim;

![docker ps](/images/docker/docker-ps.png)

Gördüğünüz gibi bir sonuç çıkmadı. Ancak biz çalıştırdık çıktıyı da gördük, çıktıyı görmeyebilirdik de. Neler oldu arka planda diye biraz şüphelendik. **docker ps** ile göremememizin nedeni o container'ın artık çalışmıyor olmasıydı. Hatırlayın, image'ın amacı ekrana birşeyler yazdırmaktı sadece. Onu da yaptı ve işi bittiği için docker engine container'ın sonlandığını biliyor. Peki bunu nasıl görebiliriz?

**docker ps -a** kullanalım;
![docker ps -a](/images/docker/docker-ps-a.png)
Image adından bakarak daha önceden çalıştırılan ve status'larını görebileceğiniz bir çıktıyı görebildik.

Hemen bitmesin arka planda çalışan bir örnek yapalım;

**docker run -d redis** kullanalım;
![docker run redis](/images/docker/docker-run-redis.png)

Redis image'ı tcp 6379 portundan expose ediliyor. DockerFile'a [şuradan](https://github.com/docker-library/redis/blob/6cb8a8015f126e2a7251c5d011b86b657e9febd6/3.0/Dockerfile) bakabilirsiniz.

DockerFile'da bizi ilgilendiren kısma bakalım
```
FROM debian:jessie

EXPOSE 6379
CMD [ "redis-server" ]
```

debian imajının jessie sürümünden türetilmiş, 6379 portundan dışarıya açılan ve komut olarak **redis-server** çalıştırdığını görebiliyoruz. Hala çalıştığı için de **docker ps** ile görebiliyoruz.

Çalışan container'dan birini silelim. Bunun için **docker rm containerID** komutunu kullanacağız;

![docker rm stop](/images/docker/docker-rm-stop.png)

1. İlk önce **docker ps** ile çalışan containerları listeledik.
2. **docker rm 15ff** ile aakkus/pingpongwithjava image'in id değeri 15ff.. ile silmeye çalıştık. Container ID değerinin tamamını vermenize gerek yok. İlk birkaç unique karekteri vererek de işlem yapabilirsiniz.
3. Docker engine bize dedi ki bu container çalışıyor sen önce bunu durdur sonra ben kaldırırım.
4. **docker stop 15ff** ile durdurduk bizde.
5. Adım 3'deki komutu tekrar çalıştırdığımızda container silinmiş oldu.

Önerilmez ama durdurmadan da silmek istiyorum derseniz **-f** parametresi ile docker rm komutunu kullanabilirsiniz;
![docker rm stop](/images/docker/docker-rm-f.png)

Bu yazıda Docker'a giriş yaptık, Docker CLI'dan bahsettik. Docker image, container gibi konulara giriş yaptık.

Bir sonraki yazıda görüşmek dileğilye.

> Alican Akkus.
