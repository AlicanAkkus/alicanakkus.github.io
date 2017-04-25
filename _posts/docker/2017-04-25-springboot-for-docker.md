---
layout: post
title: Spring Boot for Docker
permalink: /blog/docker/springboot-for-docker
summary:  Merhabalar, bu yazımızda spring boot uygulamamızı docker'da nasıl çalıştırırız ona bakacağız.
---

Merhabalar, bu yazımızda spring boot uygulamamızı docker'da nasıl çalıştırırız ona bakacağız.

### Movie Application
Uygulamaya git reposu üzerinden [şurada](https://github.com/AlicanAkkus/spring-boot-for-docker) erişebilirsiniz.

Uygulamamız kabaca bir film arama uygulaması. http://www.omdbapi.com adresinin sağlamış olduğu api ile film sorgulama yapıyoruz sadece. Çıktı aşağıdakine benzer olacaktır;

![spring-boot-for-docker](/images/docker/spring-boot-docker.png)

Uygulamayı git reposundan incelersiniz. Biz asıl konumuz olan Docker ile ilgili kısma geçelim. Dockerfile'a bakalım;

```
FROM java:8
VOLUME /tmp
ADD target/demo-0.0.1-SNAPSHOT.jar movie.jar
RUN bash -c 'touch /movie.jar'
EXPOSE 8080
ENTRYPOINT ["java","-jar","/movie.jar"]
```
Image'ımız java:8'den miras alıyor. Volume olarak yani datalarımızı /tmp altında tutuyoruz. **ADD** komutu ile proje altındaki jar dosyamızı **movie.jar** olarak image içerisinde oluşturduk. **8080** portundan expose ederek **java -jar movie.jar** diyerek uygulamayı başlatmış olduk.

**docker build -t aakkus/movie .** ile image'ı build edelim.

```
root@alican-laptop:/home/alican/examples/docker-demo# docker build -t aakkus/movie .
Sending build context to Docker daemon   24.1MB
Step 1/6 : FROM java:8
 ---> d23bdf5b1b1b
Step 2/6 : VOLUME /tmp
 ---> Using cache
 ---> 134ab173f75a
Step 3/6 : ADD target/demo-0.0.1-SNAPSHOT.jar movie.jar
 ---> 11e0ffe4617c
Removing intermediate container aa2fa291e2e3
Step 4/6 : RUN bash -c 'touch /movie.jar'
 ---> Running in b02dc48d720b
 ---> 88a4f62b28c6
Removing intermediate container b02dc48d720b
Step 5/6 : EXPOSE 8080
 ---> Running in 56a703f5da87
 ---> c7da9533ecca
Removing intermediate container 56a703f5da87
Step 6/6 : ENTRYPOINT java -jar /movie.jar
 ---> Running in 1e6d27601ebd
 ---> c3feed08d1f2
Removing intermediate container 1e6d27601ebd
Successfully built c3feed08d1f2
root@alican-laptop:/home/alican/examples/docker-demo# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
aakkus/movie                       latest              c3feed08d1f2        7 seconds ago       687MB
aakkus/my-redis                    latest              0d466e451b64        24 hours ago        183MB
aakkus/my-redis                    <none>              524fb1742e57        24 hours ago        183MB
aakkus/curl                        latest              e9aaf778fe3e        3 days ago          186MB
microsoft/mssql-server-linux       latest              ba95cfb67ac1        6 days ago          1.33GB
mysql                              latest              d5127813070b        13 days ago         407MB
aakkus/iyzico-challenge            latest              63f7b5a24734        2 weeks ago         734MB
iyzico-challenge                   latest              63f7b5a24734        2 weeks ago         734MB
aakkus/selam-docker                latest              9a77937b4106        3 weeks ago         129MB
aakkus/pingpongwithjava            latest              ebd5d61a6b44        7 weeks ago         643MB
```
Image'ı başarıyla oluşturduk. Şimdi ise run edelim;
```
root@alican-laptop:/home/alican# docker run -d -p 8080:8080 aakkus/movie
d0a6bf08b4cec12bf081d5c7293faa2c90ef0c1bdb26ab217ea358ca2e42d907
root@alican-laptop:/home/alican#
```
Çalışıyor mu ona bakalım;
```
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d0a6bf08b4ce        aakkus/movie        "java -jar /movie.jar"   28 seconds ago      Up 27 seconds       0.0.0.0:8080->8080/tcp   elastic_haibt
root@alican-laptop:/home/alican#
```
Host'da ki 8080'den container'daki 8080'e doğru bir mapping var. Tarayıcıdan localhost:8080 adresine giderek erişebiliriz.

Spring metrics'e erişmeye çalışalım;
```
root@alican-laptop:/home/alican/examples/docker-demo# curl localhost:8080/manage/metrics
{
  "mem" : 459374,
  "mem.free" : 372978,
  "processors" : 4,
  "instance.uptime" : 124123,
  "uptime" : 129816,
  "systemload.average" : 1.4,
  "heap.committed" : 406528,
  "heap.init" : 124928,
  "heap.used" : 33549,
  "heap" : 1753088,
  "nonheap.committed" : 54168,
  "nonheap.init" : 2496,
  "nonheap.used" : 52847,
  "nonheap" : 0,
  "threads.peak" : 23,
  "threads.daemon" : 19,
  "threads.totalStarted" : 26,
  "threads" : 21,
  "classes" : 6420,
  "classes.loaded" : 6420,
  "classes.unloaded" : 0,
  "gc.ps_scavenge.count" : 8,
  "gc.ps_scavenge.time" : 137,
  "gc.ps_marksweep.count" : 2,
  "gc.ps_marksweep.time" : 202,
  "httpsessions.max" : -1,
  "httpsessions.active" : 0
}
root@alican-laptop:/home/alican/examples/docker-demo# curl localhost:8080/manage/health
{
  "status" : "UP",
  "diskSpace" : {
    "status" : "UP",
    "total" : 109854089216,
    "free" : 23456473088,
    "threshold" : 10485760
  }
}
root@alican-laptop:/home/alican/examples/docker-demo#
```
Container'a bağlanalım;
```
root@alican-laptop:/home/alican# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d0a6bf08b4ce        aakkus/movie        "java -jar /movie.jar"   28 seconds ago      Up 27 seconds       0.0.0.0:8080->8080/tcp   elastic_haibt
root@alican-laptop:/home/alican# docker exec -it d0a6bf08b4ce bash
root@d0a6bf08b4ce:/# pwd
/
root@d0a6bf08b4ce:/# cd /
root@d0a6bf08b4ce:/# ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  movie.jar  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@d0a6bf08b4ce:/# cd tmp/
root@d0a6bf08b4ce:/tmp# ls
hsperfdata_root  tomcat-docbase.9104758411988217824.8080  tomcat.5985442616978849313.8080
```
Spring'in loguna gidelim;
```
root@alican-laptop:/home/alican# docker logs d0a6bf08b4ce

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.3.RELEASE)

2017-04-25 08:46:00.131  INFO 1 --- [           main] com.caysever.DemoApplication             : Starting DemoApplication v0.0.1-SNAPSHOT on d0a6bf08b4ce with PID 1 (/movie.jar started by root in /)
2017-04-25 08:46:00.136  INFO 1 --- [           main] com.caysever.DemoApplication             : No active profile set, falling back to default profiles: default
2017-04-25 08:46:00.671  INFO 1 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@5a2e4553: startup date [Tue Apr 25 08:46:00 UTC 2017]; root of context hierarchy
2017-04-25 08:46:02.886  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2017-04-25 08:46:02.905  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
2017-04-25 08:46:02.906  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.14
2017-04-25 08:46:03.003  INFO 1 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2017-04-25 08:46:03.003  INFO 1 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2339 ms
2017-04-25 08:46:03.266  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Mapping servlet: 'dispatcherServlet' to [/]
2017-04-25 08:46:03.270  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'metricsFilter' to: [/*]
2017-04-25 08:46:03.271  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2017-04-25 08:46:03.271  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2017-04-25 08:46:03.271  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2017-04-25 08:46:03.271  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2017-04-25 08:46:03.271  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'webRequestLoggingFilter' to: [/*]
2017-04-25 08:46:03.271  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'applicationContextIdFilter' to: [/*]
2017-04-25 08:46:03.993  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@5a2e4553: startup date [Tue Apr 25 08:46:00 UTC 2017]; root of context hierarchy
2017-04-25 08:46:04.088  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/],methods=[GET]}" onto public java.lang.String com.caysever.controller.HomeController.home()
2017-04-25 08:46:04.093  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2017-04-25 08:46:04.094  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2017-04-25 08:46:04.134  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-04-25 08:46:04.135  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-04-25 08:46:04.199  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-04-25 08:46:04.834  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/trace || /manage/trace.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.835  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/health || /manage/health.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.HealthMvcEndpoint.invoke(javax.servlet.http.HttpServletRequest,java.security.Principal)
2017-04-25 08:46:04.838  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/loggers/{name:.*}],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.LoggersMvcEndpoint.get(java.lang.String)
2017-04-25 08:46:04.838  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/loggers/{name:.*}],methods=[POST],consumes=[application/vnd.spring-boot.actuator.v1+json || application/json],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.LoggersMvcEndpoint.set(java.lang.String,java.util.Map<java.lang.String, java.lang.String>)
2017-04-25 08:46:04.839  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/loggers || /manage/loggers.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.840  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/metrics/{name:.*}],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.MetricsMvcEndpoint.value(java.lang.String)
2017-04-25 08:46:04.840  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/metrics || /manage/metrics.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.841  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/auditevents || /manage/auditevents.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public org.springframework.http.ResponseEntity<?> org.springframework.boot.actuate.endpoint.mvc.AuditEventsMvcEndpoint.findByPrincipalAndAfterAndType(java.lang.String,java.util.Date,java.lang.String)
2017-04-25 08:46:04.841  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/autoconfig || /manage/autoconfig.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.845  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/env/{name:.*}],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EnvironmentMvcEndpoint.value(java.lang.String)
2017-04-25 08:46:04.845  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/env || /manage/env.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.846  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/info || /manage/info.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.847  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/dump || /manage/dump.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.848  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/configprops || /manage/configprops.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.848  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/mappings || /manage/mappings.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.849  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/heapdump || /manage/heapdump.json],methods=[GET],produces=[application/octet-stream]}" onto public void org.springframework.boot.actuate.endpoint.mvc.HeapdumpMvcEndpoint.invoke(boolean,javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse) throws java.io.IOException,javax.servlet.ServletException
2017-04-25 08:46:04.850  INFO 1 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/manage/beans || /manage/beans.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2017-04-25 08:46:04.951  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2017-04-25 08:46:04.963  INFO 1 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 0
2017-04-25 08:46:05.109  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-04-25 08:46:05.115  INFO 1 --- [           main] com.caysever.DemoApplication             : Started DemoApplication in 5.395 seconds (JVM running for 6.115)
2017-04-25 08:48:08.753  INFO 1 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring FrameworkServlet 'dispatcherServlet'
2017-04-25 08:48:08.754  INFO 1 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization started
2017-04-25 08:48:08.774  INFO 1 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization completed in 20 ms
root@alican-laptop:/home/alican#
```

Herşey yolunda görünüyor. Docker hub'dan çekmek için şunu kullanabilirsiniz;
**docker pull aakkus/movie**

Yazıymızı burada sonlandırıyoruz, bir sonraki yazımızda Docker Network konusuna değineceğiz.

> Alican Akkus.
