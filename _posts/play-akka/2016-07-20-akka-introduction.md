---
layout: post
title: Akka Introduction
permalink: /blog/java-platform/play-akka/akka-introduction
summary: Akka ile beraber concurrent mekanizmaları, paralel hesaplamalar, dağıtık mimarilerde efektif iş yapma gibi konulardan bahsedecegiz.
image: images/java-platform/java-ee/jms/jms_logo.png
---

Merhaba arkadaslar, yeni bir yazı serisine bu yazı ile başlamış bulunmaktayım.

Akka ile beraber concurrent mekanizmaları, paralel hesaplamalar, dağıtık mimarilerde efektif iş yapma gibi konulardan bahsedecegiz. Akka mimarisinden, concurrent'ı nasıl ele aldıgından, nasıl modellediginden bahsedecegiz. Bu yazımızda ise Akka ve bazı kavramlara giriş yapacağız.

Detaylı bilgi için sitesini ziyaret edebilirsiniz : <a href="http://akka.io"> http://akka.io</a>

Ayrıca Reactive manifestosuna da göz gezdirebilirsiniz : <a href="http://www.reactivemanifesto.org">http://www.reactivemanifesto.org</a>
<h3>Terminology</h3>
Akka'ya başlamadan önce bazı terminolojileri, konsept ve kavramları bilmek faydalı olacaktır. Bunlar;

* Sync ve Async.
* Reactive.
* Blocking, Non Blocking.
* Concurrency, Deadlock.

### Akka
Akka temelde concurrent haberleşmelerde, dağıtık sistemlerde, message-driven, event-driven modele sahip Scala ile yazılmıs bir JVM toolkit'tir. Bir framework değildir var olan uygulamalara/projelere kolayca adapte edilebilir.

Neden Akka diye sorarsanız muhtelemen şöyle bir cevap verebiliriz; Akka actor model ile birlikte tüm haberleşme(sistemler/threadler/uygulamalar) altyapısını soyutlayarak çok basite indirger. Yaptığı soyutlama ile concurrent uygulamalar, paralel hesaplamalar vb bir çok aslında zor olan modellemeler oldukça kolaylaşmakta. Basite indiger cümlesinden kastımız ise basit işler yapmak değil, zor ve karmaşık olan işlemleri soyutlayarak sizi sadece uygulamanızın logic'i ile uğraşmanızı sağlamasıdır.

Örneğin Java'da Thread'leri yönetmek zordur. Belki de core Java içerisinde benim en çok önem verdiğim konu da Thread'lerdir. Java bize zengin bir api sunmasına rağmen Thread yönetimi karmaşık ve zordur. Dağıtık mimarileri gerçekleyebilmek, ölçeklenebilir sistemler ile birlikte failure durumlarında sistemin çalışmasını sürdürebilir durumda olması, concurrent gibi konular gerçekten zor konulardır ve kafa yorulması gereken konulardır. Akka bize bu konuda yardımcı olacağını vaad ediyor.

Akka, Scala ile yazılmıs olmasına rağmen JVM üzerinde koştuğu için Java ile de birlikte çalışabiliyor. Bu sayede Java biliyor iseniz Scala, Scala biliyor iseniz Java öğrenmek zorunda kalmayacaksınız.

Son zamanlarda Java ve/veya Scala ile Play/Akka kullanarak high performance, scalable uygulamalar yazmak oldukça çekici geliyor. Veri büyüdükçe veriye erişim ve kullanma zorlaşacağından dolayı gelecekte eş zamanlı işler yapabilen, reactive olan ve concurrent olabilen tüm ürün/araç vs daha önemli olacaktır.

### Actors

Akka, concurrent'i saglayabilmek için Actor Model'ini gerçekler/implemente eder. Yaptığı soyutlama ile Thread'lerin olusturulması, schedule edilmesi, Thread'ler arası haberleşme, mesajlaşma, senkronize olma durumları basite indirgenir. Actor model'de her şey bir actor'dur. Actor, kendisine gelen uyarı/event/message için local decision yapabilir(hesaplama, persist, ws call vs), farklı actor'ler olusturabilir, geri mesaj gönderebilir, mesajı ignore edebilir. Actor based bir sistem içerisinde tüm soyutlamalar yapılır. Actor based sistemde her şey bir actor'dur, object-oriented bir dilde herşeyin bir obje olması gibi.

Herhangi bir Actor bilinen bir objeden farksız degildir. Kabaca mesaj alır/verir. Bunu yaparken ise actor ile sistem arasında bir katman oluşturulur, bu katman ile birlikte tüm kompleks karmaşıklık Akka toolkit tarafından soyutlanır. İlerde görecegiz ne kadar basite indirgediğini.

Actor'lerin belli başlı karekteristikleri/davranışları vadır;
* Asenkron olarak iletişimi sağlar, method call ile değil.
* Kendi state'ini kendisi düzenler/günceller.
* Child actor oluşturabilir.
* Diğer bir actor ile konuşabilir.
* Kendini durdurabilir.

Actor yapısını ortaya atan abimiz şurada -> [Carl Hewitt](https://en.wikipedia.org/wiki/Carl_Hewitt#Actor_model)

Kendisi Actor model üzerinde uzun uzun yıllar çalışmış ve kafa yormuş biridir. İlham alınabilecek kişilerdendir.

Actor modelinin çözdüğü belli başlı problemler var. Bunların başında geleneksel OOP mantığında bulunan bir takım kısıtlamalardır. object-oriented programming de obje(adı O1 olsun) üzerinde bir method çağrılır, adı methodA olsun. methodA methodu da O2 objesi üzerinde methodB methodunu çağırıyor olsun. Bu durumda O1 üzerinde ki methodA methodunun çalışıp sonlanması için O2 üzerinde ki methodB methodunun da çalışıp sonlanması gerekir. Design pattern kavramında ki loose coupling/tightly coupling programlama seviyesine kadar inmiş olur.

![oop method call](/images/play-akka/method-call.png)

Pure OOP ile zaten karmaşık ve zor olan yapı multi-thread olan uygulamalar ile yönetilmesi oldukça zor olan yapılara doğru evriliyorlar. Akka ise Actor model ile birlikte bu işi kolaylaştırır. En basit anlatımı ile her actor birbiri ile mesaj göndererek iletişime geçer. OOP da ise lock mekanizmaları, senkronize bloklar vs tonla yönetilmesi gereken iş vardır.

Akka ile örnek olması açısından aynı yapıya bakalım;

![akka call](/images/play-akka/akka-call.png)

Yazımızı burada sonlandırıyoruz, sonraki yazılarda Actor-Based sistemler, Actor'ler, Java ve Actor Thread modellerinin kıyaslanması gibi konulara değinmeye çalışacağız.

Mutlu ve esen kalın.

> A.Akkus
