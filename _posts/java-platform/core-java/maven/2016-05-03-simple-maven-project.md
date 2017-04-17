---
layout: post
title: Simple Maven project
permalink: /blog/java-platform/core-java/quartz/simple-maven-project
summary: Merhaba arkadaslar, bu yazımızda örnek bir maven projesi oluşturup build etmeyi göreceğiz.
image: images/java-platform/java-ee/jms/jms_logo.png
---
Merhaba arkadaslar, bu yazımızda örnek bir maven projesi oluşturup build etmeyi göreceğiz.

Eclipse üzerinde new Maven Project diyelim ve başlayalım;

Adım - 1

![maven step 1](/images/java-platform/core-java/maven/mvn1.png)

Create a simple project box'unu seçmeden devam edelim, bir sonraki adımda archetype seçecegiz.

Adım - 2

![maven step 2](/images/java-platform/core-java/maven/mvn2.png)

Maven projesi oluştururken hazır olarak gelen birçok proje template'i bulunmaktadır. Quick start olması için ilgili archetype'ı seçip devam edelim;

Adım - 3

![maven step 3](/images/java-platform/core-java/maven/mvn3.png)

Bu adımda projemiz ile ilgili bilgileri verecegiz maven'e.

* Group Id : Uygulamanın domanini/grubunu belirler. Oluşturulacak olan paketler com.wora altında olacaktır.
* Artifact Id : Uygulamanın adı olacaktır. Group Id + Artifac Id ile unique bir isim yakalıyor olmamız lazım. Java'da tersine paket isimlendirmesi de bundan dolayıdır.
* Version : Default olarak 0.0.1-SNAPSHOT olarak gelir devam edelim.


Adım - 4

![maven step 4](/images/java-platform/core-java/maven/mvn6.png)

Maven bizim için paketleri ve App class'ını oluşturdu. pom.xml'e ve App.java'ya bakalım;

pom.xml;
``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.wora</groupId>
	<artifactId>MavenTutorial</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>MavenTutorial</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> <!-- UTF-8 ile build et -->
	</properties>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
</project>
```

Proje oluşturma adımında girdigimiz değerleri pom.xml'de görebiliriz. Packaging default olarak jar'dır. Maven'e bu uygulamayı paketle dediginizde ilgili tip için export alırsınız.

App.java
``` java
package com.wora.tutorial;

/**
 * Hello world!
 *
 */
public class App {
	public static void main(String[] args) {
		System.out.println("Hello World!");
	}
}
```
Adım - 5

![maven package step](/images/java-platform/core-java/maven/mvn-package.png)

Proje'ye sağ click ve Run Maven Build diyelim. Goals kısmına packaging yazalım. Bu ifade, consolda "mvn package" ile aynıdır. Run diyelim ve consol çıktısına bakalım;

![maven package step finished](/images/java-platform/core-java/maven/mvn4.png)

Consolda build succes ifadesini gördük. İlk paketleme biraz network'den dolayı da uzun sürebilir. Repositorileri download ettikten sonra daha hızlı çalışacaktır. Repositoriler m2 dizini altındaki repository klasörüne indirilir.

Jar dosyasını "target" altına Artifact Id + Version + packaging type isimlendirmesinde çıkarmış oldu. Çalıştırıp bakalım;

![maven step 5](/images/java-platform/core-java/maven/mvn5.png)

Not : java -jar yada java -cp farketmez çalıştırabilirsiniz. Manifest dosyası yoksa -cp ile classpath bilgisini vererek run etmelisiniz.

Aşağıdaki linkten ilgili projeye erişebilirsiniz. Eclipse yada farklı bir ide'de import maven project demeniz yeterli. Maven, repositorileri vs kullanıma hazır hale getirecektir. Maven, taşınabilirliği, bağımlılıları hallediyor dedik ya hah iste tam sırası. Projenin dosya boyutu sadece 121 kb.

Proje : [MavenTutorial](https://www.dropbox.com/sh/ji4f9fprnpwlziu/AAAzX34epTg-8QLBT4MJ2wvTa?dl=0)

Yazımızın sonuna geldik, bir sonraki yazıda pom.xml'e detaylıca değinicez ve asıl alanımız olan java web application oluşturacağız.

Mutlu ve esen kalın.

> A.Akkus
