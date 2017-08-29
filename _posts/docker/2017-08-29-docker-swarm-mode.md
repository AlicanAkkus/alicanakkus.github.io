---
layout: post
title: Docker Swarm Mode Introduction
permalink: /blog/docker/docker-swarm-mode-introduction
summary: Merhabalar, bu yazımızda Docker Swarm Mode üzerine konuşacağız. Yazımıza Docker Swarm Mode kavramının ne olduğundan bahsederek başlayacağız. Ardından use case'lere bakıp, işin teorisine göz atacağız.

---

Merhabalar, bu yazımızda Docker Swarm Mode üzerine konuşacağız.

Yazımıza Docker Swarm Mode kavramının ne olduğundan bahsederek başlayacağız. Ardından use case'lere bakıp, işin teorisine göz atacağız. Bir sonraki yazımız da ise local makinamızda swarm cluster oluşturup denemeler yapacağız.

## Docker Swarm Mode Feature
Swarm mode, Docker platformu için container orchestration aracıdır. Bildiğiniz gibi Docker ile container oluşturup işlerimizi hallediyor, uygulamamızı dockerize edip birçok problemden kurtulmuş oluyorduk. Bunun bir tık ilerisinde docker compose yapısı ile birden fazla container'ı aynı anda ve tek komut ile çalıştırabilmiştik.

Bunun da bir tık ötesi Docker Swarm olmaktadır. DB, App Server, Web Server, Cache, MQ gibi bileşenlerden oluşan uygulamanızı Docker Swarm ile yönetebilir, yük altında scale edebilir, multiple hostlar üzerinde birden fazla instance koşturarak single point of failure problemini çözebiliriz.


Docker'ın son versiyonlarında in-built olarak Swarm mode yüklüdür. Swarm mode yeteneklerine bakalım;

* **Cluster management integrated with Docker Engine** : Docker biliyor ve kullanıyor iseniz container orchestration için ayrı bir tool vs öğrenmenize gerek kalmayacaktır. Docker CLI komut setine çok benzer komutlarla container'ları yönetebilirsiniz.

* **Declarative service model** : Declarative yaklaşım ile uygulama stack'i içerisinde ki bileşenleri gruplayabilir ve birbirinden ayırabilirsiniz. Örn; MQ, Redis gibi bileşenleri backend servisleri olarak ayarlayabilir, Web Server vb bileşenleri ise frontend servisleri olarak ayırabilirsiniz. Bu özelliğin şu avantajı olabilir; servisleri gruplayarak frontend ile gruplanmış bileşenlerin direkt olarak backend servisleri ile iletişim kurmalarını engelleyebilirsiniz yada izin de verebilirsiniz.

* **Scaling** : Uygulama stack'i içerisinde ki herhangi bir servisi scale edebilirsiniz. Docker Swarm'a sadece ilgili servis için kaç tane instance çalışacağını belirtmeniz yeterli olacaktır.

* **Desired state reconciliation** : Docker Swarm, beklenen state ile mevcut state'i sürekli karşılaştırıp bunların birbirini doğrular seviyede olmasını sağlar. Örn; MQ servis bileşeninizden 3 adet instance ile çalışıyorsunuz. Beklenen state : 3 MQ container, Mevcut state : 3 MQ container. Olur da bir container stop oldu veya ulaşılamaz oldu, bu durumda mevcut state de 2 container olacak ve beklenen state ile mutabakatsızlık olacaktır. Docker Swarm bunu algılayarak hemen 3.cü container'ı sizin için oluşturacaktır.

* **Service discovery** : Docker Swarm içerisinde in-built olarak gelen özelliklerden biri de embedded DNS serverdır. Yine örneğimiz üzerinden ilerleyelim; APP Server bileşeniniz Swarm ortamında MQ servisine erişmek istediğinde sadece "MQ" demesi yeterlidir, ip bilgisine ihtiyaç duymazsınız. Docker Swarm, her servis için unique bir DNS kaydı oluşturacaktır.

* **Load balancing** : Yine Docker Swarm içerisinde in-built olarak gelen bir özellik de load balancing'tir. 4 Web Server instance var ise Docker Swarm bu servise gelen istekleri akıllıca bir şekilde dengeleyerek dağıtır. Ek olarak hayır Docker Swarm'ın bunu yapmasını istemiyorum derseniz de kendi load balancer(Netscaler, F5)'a bağlayabilirsiniz servislerinizi.

* **Secure by default** : Docker Swarm mode ortamında bulunan her host TLS ile haberleşir. Herhangi bir CA'dan aldığınız sertifikayı da verebilirsiniz, default olarak Swarm sizin için bir tane oluşturur.

* **Rolling updates** : 3 adet Redis 3.0.6 ile çalışan instance'ınızın olduğunu varsayalım ve version upgrade yapacaksınız. Docker Swarm size bunu kolayca sağlayacaktır. Redis 3.0.7 versiyona upgrade edilecektir her bir instance. Herhangi birinde fail olması durumunda ise rollback ile mevcut yapıya dönülecektir.

> Not : Docker Swarm da her servisin önceki versiyonu tutulur. İstediğiniz zaman manuel olarak da rollback yapabilirsiniz.

### Docker Swarm Mode Use Cases

* Docker Swarm mode ile yukarda da ifade ettiğimiz gibi containerları yönetebilir, birden fazla host'u tek bir host gibi kullanabilir, scale edebilir ve kullanabilirsiniz.

* Development'ı kolaylaştırır. Geliştiricilerin en büyük problemlerinden olan **benim makinamda çalışıyordu**'yu çözer. Örn; Prod ortamında ön tarafta bir load balancer vardır, arka tarafta birden fazla node vs vardır. Bir problem de geliştirci, prod ortamında ki yapıyı localinde gerçekleyemediği için problemi tam olarak çözemez veya zorlanır. Bu tip durumlar için environment'ların birbirine benzemesini sağlayacaktır.
Kısaca; Uygulama prod ortamında nasıl çalışıyor ise dev, sandbox ve local ortamlar da da aynı şekilde ve yapıda çalışacaktır.

### Docker Swarm Mode

Docker Swarm'da birden fazla host(node) olduğunu ifade ettik. Node'lar sahip oldukları role'e göre farklı görevleri gerçekleştirirler. Bu roller;

1. **Manager** : Manager rolünde ki node'lar docker swarm'ı yönetme yeteniğe sahiptir, swarm'a ait metadaları okuyabililir ve değiştirebilir.

2. **Worker** : Worker rolünde ki node'lar genelde servisleri koşturmakla yükümlü olan, işi yapan arkadaşlardır. Manager hangi servisin çalışacağını söyler, worker da çalıştırır.

> Not : Default olarak Manager role sahip node'lar iş yapabilirler yani servisleri üzerlerinde koşturabilirler. Ama genellikle manager'a iş yaptırmak pek doğru bir yaklaşım değildir. Peki neden default da iş yapıyor diye sorabilirsiniz :) Bana göre cevabı şu: Docker Swarm mode ile çalışabilmek için en az 1 tane manager role de bir adamın bulunması lazımdır. Worker için böyle bir zorunluluk yoktur. Yani siz swarm'ı denemek istiyorsunuz ve sadece manager node kurarak deneyebiliyorsunuz. Ekstradan ikinci bir node kurmanıza gerek kalmıyor. Manager olan adamın iş yapmamasını istersek de bu özelliği kapatabiliriz manager için.

Docker Swarm Mode mimarisine göz atalım;

![docker swarm mode](/images/docker/docker-swarm-mode.png)

Mimariye bakacak olursak Manager'ların kendi aralarında ve worker'larla konuştuğunu göreceğiz. Worker'lar kendi aralarında iletişim kurmazlar.

Manager node'lar kendi aralarında consensus(fikir birliği) sağlamak için RAFT algoritmasını kullanmaktadır. Örn; leader election gibi konular için RAFT kullanılıyor Docker Swarm içerisinde. Detatına çok girmeyeceğiz ama **RAFT** bize şu matematik ile cluster ortamınızın sağlıklı çalışacağını ifade ediyor.

**(N - 1) / 2** : Örneğin cluster ortamınızda 3 adet manager node var ise (3-1)/2 = 1, yani 1 manager node down olsa bile cluster mimariniz bundan etkilenmeyecektir.

> Not : Ne kadar çok Manager olursa swarm cluster daha iyi çalışacaktır varsayımı doğru değildir. Manager node sayısı arttıkça consensus sağlanması da uzun ve zor olacaktır. Genel de önerilen max manager node sayısı 7 olarak tavsiye edilir.

Birçok kez servis kelimesini telaffuz ettik. Docker Swarm Mode içerisinde servisler birer task olarak da görünür. Yukarda ifade ettiğimiz MQ, Redis vb bileşenler birer servistir. Bunların container olarak çalışıyor olması ise birer tasktır. Hemen aşağıda göstermeye çalışalım;

![docker swarm mode service](/images/docker/docker-service.png)

Bir nginx servis tanımımız var ve biz bundan 3 tane instance çalıştırılmasını istemişiz. **nginx** burada ki servisimiz, node'lar üzerinde çalışan **nginx.1**, **nginx.2**, **nginx.3** ise tasklarımız olmaktadır. Bu tasklardan herhangi birinin göçmesi durumunda Docker Swarm tarafından yeni task oluşturulup çalıştırılacaktır.

> Not : Birçok kez 2, 3 ve 4 gibi instance adetlerinden bahsediyoruz. Best practice olarak bir servisi silip/kaldırmak yerine ilgili servisin instance sayısını 0'a çekmek önerilir.

#### Docker Swarm Mode Service Type

Bahsetmiş olduğumuz servisler sadece swarm ortamında çalışmaktadır. Yani eğer bir swarm init etmediyseniz veya swarm'a join olmadınız ise docker engine üzerinde servise create edemezsiniz. Swarm ortamında da sadece swarm manager node üzerinde oluşturabilirsiniz. Bir sonraki yazımızda servisler ile ilgili birçok örnek yapacağız ama teorik olarak bilgi sahibi olmakta fayda var.

Servisler 2 tip olarak karşımıza çıkarlar. Global type and Replicated type.

* **Global service** : Global servisler, swarm ortamında bulunan tüm nodelar(manager & worker) da çalışan servislerdir ve instance adeti verilemez. Her yeni eklenen node için de bu servis default olarak çalıştırılır. Her node üzerinde çalışmasını istediğiniz monitoring agents, anti-virus scanners gibi servisleriniz için global type tanımı yapabilirsiniz.

* **Replicated service** : İlgili servisten kaç adet instance'ın çalışacağına sizin karar verdiğiniz servis tipidir.

İki tipten birini seçmek oldukça kolaydır, service create ederken **--mode** parametresi ile belirleyebilirsiniz. Sonraki yazıda örneğini yapacağız.

Yazı ile ifade ettiklerimizi resmedelim;

![docker swarm mode service](/images/docker/docker-replica-and-global-services.png)

Docker Swarm Mode yazımızı burada sonlandırıyoruz, bir sonraki yazıda kendi swarm ortamımızı oluşturup servisler çalıştıracağız.

> Alican Akkus.
