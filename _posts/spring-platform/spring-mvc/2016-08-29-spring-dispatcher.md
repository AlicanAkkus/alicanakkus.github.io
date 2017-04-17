---
layout: post
title: Spring Dispatcher
permalink: /blog/spring-platform/spring-mvc-tutorial/spring-mvc-dispatcher
summary: Merhaba arkadaşlar, bu yazımda Spring Dispatcher tanımına bakacağız.
image: images/spring-platform/spring-mvc/springmvc.png
---

Bir önceki yazımda Spring MVC mimarisinin en önemli parçası olan ve request lifecylce'ının ilk adımı oldugundan bahsettik.

Spring MVC, Front Controller design pattern ile birlikte request based bir yaklaşımı kullanmaktadır. Front Controller'ın avantajı, tüm requestler merkezi bir yerde toplanır ve tek bir noktadan giriş yapar. Dispatcher Servlet, Spring için olmazsa olmaz temel taşlardan biridir.

Dispatcher Servlet temelde pure bir Servlet'tir ve HttpServlet'den kalıtılmıştır. Ancak pure Servlet olmasının yanı sıra Spring IoC Contaıner özelliklerini ve Spring'in diğer özelliklerini de kapsamaktadır.

Spring DispatcherServlet şeması aşağıdaki gibidir;

![spring dispatcher](/images/spring-platform/spring-mvc/spring-mvc-dispatcher1.png)

![spring dispatcher](/images/spring-platform/spring-mvc/spring-mvc-dispatcher2.png)

Yukarıdaki iki görsel tüm Spring MVC mimarisini bir bakışta anlamanızı sağlayabilir.

DispatcherServlet tanımı deployment descriptor olan Java web uygulamalarının konfigürasyon dosyası yani web.xml içerisinde standart bir Servlet gibi tanımlanır. Tanımlanan servlet için url mapping uygulanır ve hangi tip isteklerin Spring tarafından handle edilmesi istenildiği belirtilir.

Yukarıdaki iki görsel de de request lifecycle'ın başlangıç noktası Dispacher Servlet'tir. Diğer modüller olan Handler Mapping, Controller, View Resolver, View katmanları request lifecyle içerisindeki diğer aşamalardır. Bir önceki yazıda belirttiğim gibi Dispatcher Servlet isteği yakalar ve Handler Mapping yardımı ile gerekli Controller'ı bulmaya çalışır, Annotation'larla işaretlediğimiz class/methodlar yani. Controller bulunduktan sonra request buraya aktarılır ve gerekli logic'ler yapılır. Controller'dan view işaretçisi iletilir ve View Resolver ile hangi view'ın render edileceği bulunur. Bulunduktan sonra model bind edilebilir ve/veya view response olarak dönülebilir.

Örnek bir DispatcherServlet tanımına bakalım;

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0">
  <display-name>CrunchifySpringMVCTutorial</display-name>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>

</web-app>
```

web.xml'de Servlet class olarak DispatcherServlet verdik ve url mapping olarak **"*.do"** olarak mapledik. Spring default olarak WEB-INF altında(web.xml orada bulunduğu için) servletname-servlet.xml adında bir dosya arar. Bu dosya bean definition olarak tanımlansa da birçok konfigürasyon buradan ayağa kaldırılır. Yukarıdaki örnekte WEB-INF altında springmvc-servlet.xml dosyasının bulunması gereklidir. Dosya bulunamaz ise Spring, BeanDefinitionStoreException fırlatır. İsimlendirme standartı olarak servlet adı + "-" +servlet + ".xml" şeklinde bir standart mevcut. Default olarak yukarıda belirttigim sekilde arar. Ancak farklı bir yere konumlandırabiliriz. Bunun için context param ve/veya init param şeklinde ekleyebiliriz.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0">
  <display-name>CrunchifySpringMVCTutorial</display-name>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>

    <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>
           /WEB-INF/config/springmvc-servlet.xml
           /WEB-INF/security/spring-security-context.xml
       </param-value>
     </context-param>
</web-app>
```

contextConfigLocation ile bean definitiaonları load edebiliriz. Birden fazla xml file load edilebilir. init-param da aynı şekildedir sadece servlet tag'leri arasına yazılır, Servlet bilenler hemen anlayacaktır.

Bean definitionlar ile beraber aslında DispatcherServlet'in sadece merkezi bir servlet olup request handle etmedigini görmüs olduk. Örnek bean definitionumuza bakalım;
``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan base-package="com.wora.controller" />

	<bean id="viewResolver"
		class="org.springframework.web.servlet.view.UrlBasedViewResolver">
		<property name="viewClass"
			value="org.springframework.web.servlet.view.JstlView" />
		<property name="prefix" value="/jsp/" />
		<property name="suffix" value=".jsp" />
	</bean>

</beans>
```

Bean definition olarak viewResolver verdik ilk aşamada. Bunun detayına belki daha sonra inebiliriz. <context:component ile Spring'e Annotation'larla derdimizi anlatacağımızı belirttik. com.wora.controller paketi altında çalışacaktır dispatcher'ımız. View resolver'da prefix ve suffix olarak WebContent altında ki jsp klasörü içerisindeki .jsp uzantılı dosyaları kullanacağız dedik. Request lifecyle son aşamalarında ViewResolver yardımı ile ilgili view bulunur ve response olarak return edilir.

Controller'a bakalım;
``` java
package com.wora.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;


@Controller
public class BaseController {

	@RequestMapping("/")
	public String home() {
		return "index";
	}

}
```

Oldukça kısa ve sasde bir pure Java class'ından ibaret. Context name'den sonra örneğin localhost:8080/springmvc yazıldığında home() controller çalışacaktır. **@Controller** ile class'ın Spring Controller'ı oldugunu belirttik. **@Controller**, class level'dır, method level'da kullanılamaz. RequestMapping ile  de home page gösterilecek. **@RequestMapping** hem class level hem method level'dir. Bu sayede class level'da da url tanımı yapılabilir. Dikkat ederseniz return type olarak String ve değer olarak da "index" döndürdük. Bu, bak Spring kardeş ben sana view name olarak index'i verdim git viewResolver yardımıyla(suffix ve prefix) view'ı bul ve response olarak dön. Örnegimizde az önceki url girildiğinde **/WebContent/jsp/index.jsp** sayfası view olarak görüntülenecektir.

Yeri geldikçe detaylandıracağımız konular olacaktır. Bu yazıyı burada sonlandırıyorum arkadaslar.

Mutlu ve şen kalın.

> A.Akkus
