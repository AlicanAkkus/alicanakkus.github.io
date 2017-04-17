---
layout: post
title: Quartz Configuration
permalink: /blog/java-platform/core-java/quartz/quartz-configuration
summary: Merhaba arkadaslar, bu yazımda Quartz'ın configürasyon dosyalarından ve içeriğinden bahsediyor olacağım.
image: images/java-platform/java-ee/jms/jms_logo.png
---

Merhaba arkadaslar, bu yazımda Quartz'ın configürasyon dosyalarından ve içeriğinden bahsediyor olacağım.

Quartz'ın, Java'da tanımlanmış görevlerin zamanında çalışmasını sağlayan bir proje oldugunu söylemistik. Core Java Timer'daki farklarına deginmistik, bunlar cluster olarak çalıabilmesi, persistent olması vs idi.

Bugün quartz'ı configüre etmeyi göreceğiz. Quartz temelde iki configürasyon dosyası üzerinden çalışır. Bunlardan biri xml diğeri ise properties uzantılı dosyalardır.

* Quartz.xml : Job tanımlamalarının yapıldığı bölümdür.
* Quartz.properties : Quartz'ın çalışabilmesi için gerekli olan meta data bilgilerini içerir.

Şimdi örnek bir quartz.xml hazırlayalım;
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<job-scheduling-data
	xmlns="http://www.quartz-scheduler.org/xml/JobSchedulingData"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.quartz-scheduler.org/xml/JobSchedulingData
    http://www.quartz-scheduler.org/xml/job_scheduling_data_2_0.xsd"
	version="2.0">

	<schedule>
		<job>
			<name>HesapHasenât</name>
			<group>Hesap</group>
			<description>Müminin hayır islerini hesap eder.</description>
			<job-class>mumin.hesap.HesapHasenâtTrigger</job-class>
			<durability>true</durability>
			<recover>false</recover>
		</job>
		<trigger>
			<cron>
				<name>HesapHasenâtTrigger</name>
				<job-name>HesapHasenât</job-name>
				<job-group>Group1</job-group>
				<!-- 12 VE 24 de calis -->
				<cron-expression>0 0 12,24 * * ?</cron-expression>
			</cron>
		</trigger>
	</schedule>

	<schedule>
		<job>
			<name>HesapSeyyihât</name>
			<group>Hesap</group>
			<description>Müminin şer islerini hesap eder.</description>
			<job-class>mumin.hesap.HesapSeyyiâtTrigger</job-class>
			<durability>true</durability>
			<recover>false</recover>
		</job>
		<trigger>
			<cron>
				<name>HesapSeyyihâtTrigger</name>
				<job-name>HesapSeyyihât</job-name>
				<job-group>Group1</job-group>
				<!-- 12 VE 24 de calis -->
				<cron-expression>0 0 12,24 * * ?</cron-expression>
			</cron>
		</trigger>
	</schedule>

</job-scheduling-data>
```

Quartz.xml içerisinde root node job-scheduling-data'dır. Yani bu dosya aslında jobları ve jobların çalışma zaman tanımlarını içerir.

Job tanımı yapmamız için bir schedule, zamanlama oluşturmamız gerekiyor. Schedule, içerisinde job bilgisini ve ne zaman çalışağını içerir.

Job tanımına bakalım;

* name : Alias gibi düsünebilirsiniz, job'a isim atama.
* group : Jobları gruplayabilirsiniz. Ben **"Hesap"** olarak grupladım.
* description : Job detayı için kullanabiliriz.
* job-class : Job'ın çalışma zamanı geldiğinde tetiklenecek olan sınıfı belirler, birazdan bakacağız.
* durability : Herhangi bir anda job'ı trigger eden bir islem yok ise hafızada tutulsunmu değeridir. Job tanımı yapıp trigger etmeyebilirsiniz anlamı cıkar buradan. Biz true set ettik.
* recover : Job'ın trigger anında herhangi bir hata oluşursa fail-over işlemi yaparak re-execute yapmasını sağlar.

Trigger tanımına bakalım;

* cron : Job'ın ne zaman tetikleneceğini ifade eder.
* name : Cron'a isim verilebilir.
* job-name : Job'un adı.
* job-group : Job'un bulunduğu grup.
* cron-expression : Job'ın çalışma zamanlamasını belirtir. Bir expression'dur, örnek expressionlara yer verecegim yazının devamında. Örnekte günde iki kere 12 ve 24'de çalışmasını istemişiz.

Job tanımlarını ve Trigger tanımlarını yaptıktan sonra quartz'ın çalışması için gerekli olan metadata bilgilerini içeren properties dosyasına bakalım;
``` java
#======================
# Created by Caysever
#======================


#============================================================================
# Configure Main Scheduler Properties
#============================================================================

org.quartz.scheduler.instanceName = AmelScheduler
org.quartz.scheduler.instanceId = AUTO
org.quartz.plugin.jobInitializer.class =org.quartz.plugins.xml.XMLSchedulingDataProcessorPlugin
org.quartz.plugin.jobInitializer.fileNames = quartz.xml
org.quartz.plugin.jobInitializer.failOnFileNotFound = true


#============================================================================
# Configure ThreadPool
#============================================================================

org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount = 5
org.quartz.threadPool.threadPriority = 5

#============================================================================
# Configure JobStore
#============================================================================

org.quartz.jobStore.misfireThreshold = 60000
#org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.PostgreSQLDelegate
org.quartz.jobStore.useProperties = false
org.quartz.jobStore.dataSource = myDS
org.quartz.jobStore.tablePrefix = QRTZ_

org.quartz.jobStore.isClustered = true
org.quartz.jobStore.clusterCheckinInterval = 20000

#============================================================================
# Configure Datasources
#============================================================================
#org.quartz.dataSource.myDS.jndiURL=jdbc/caysever
org.quartz.dataSource.myDS.jndiURL= java:/comp/env/jdbc/caysever
org.quartz.dataSource.myDS.maxConnections = 5
org.quartz.dataSource.myDS.validationQuery=select getDate()


```

Properties dosyasını inceleyelim;

1. Main ayarlara bakalım;
* **org.quartz.scheduler.instanceName =** Oluşturuğumuz scheduler dosyamız için bir isim veriyoruz.
* **org.quartz.scheduler.instanceId** = Scheduler intance'a auto id atadık.
* **org.quartz.plugin.jobInitializer.class** = Job'ları xml ile belirttiğimizi söyledik.
* **org.quartz.plugin.jobInitializer.fileNames** = Job'ları hangi dosyada oldugunu söyledik. Default olarak aynı dizine koymayı tercih ediyorum.
* **org.quartz.plugin.jobInitializer.failOnFileNotFound** = Job dosyası bulmadığında fail olsun ve başlamasın diye true set ettik.

> Not : Instance Name, her scheduler için özeldir. Yani her scheduler'ın kendi konfigürasyon dosyası olmalıdır. Aynı configürasyon içerisinde farklı scheduler tanımalamaz.**

2. Thread Pool ayarlarına bakalım;
* **org.quartz.threadPool.class** = Job'ların yönetilecegi thread pool classtır. Simple Thread Pool kullanalım. Required bir alandır. Default'u null'dır.
* **org.quartz.threadPool.threadCount** = Count bilgisi, aynı anda kaç job'ın çalışabileceğini belirtir. Required alandır, default -1'dir. Job'ların sayısına ve çalışma sıklığına göre değeri büyütebilirsiniz. Pratikte 1 ile 100 arası kullanabilirsiniz.
* **org.quartz.threadPool.threadPriority** = Job'un çalıştığı Thread'ın priorty bilgisidir. Min ve Max priorty girilebilir. Zorunlu bir alan değildir.

3. Job Store ayarlarına bakalım;
* **org.quartz.jobStore.misfireThreshold** = Job tetiklenemediyse sonraki çalışma zamanı için tolerans değeridir. Defaultu  60sn'dir.
* **#org.quartz.jobStore.class** = Comment halde duruyor bu prperties. Job^ların nasıl saklanacağını belirtir. Memory üzerinde de tutabiliriz.
* **org.quartz.jobStore.class** = Job'ları jdbc ile persist etmesini istedik.
* **org.quartz.jobStore.driverDelegateClass **= Jdbc class'ını verdik.
* **org.quartz.jobStore.useProperties** = Bunu false olarak ayarlamak tercih edilir. True set edilirse job bilgilerini bir map'de name-value olarak tutuyor complex obje yerine.
* **org.quartz.jobStore.dataSource** = Bir datasource kullanacağımızı belirttik. Datasource tanımı yazının devamında.
* **org.quartz.jobStore.tablePrefix** = Quartz, job'ları saklamak için kendi tablolarını oluşturacaktır. Table prefix ile ön ek verebiliriz.  Quartz tarafından mesela şöyle tablolar oluşur : QRTZ_TRIGGERS , QRTZ_JOB_DETAIL , QRTZ_CALENDARS vs.
* **org.quartz.jobStore.isClustered** = Clustur olarak çalış dedik. Bu konu bambaşka bir yazı olabilecek kadar geniş ve önemli. Quartz tablolarına bakarak instance'lardan gelen job işlerini yönetir. Fail over, scalable gibi noktaları var.
* **org.quartz.jobStore.clusterCheckinInterval** = Diğer instance'ları hangi sıklıkla kontrol edecegini belirtiyoruz.

> Not : isClustered değerini, eğer birden fazla instance'ız var ve aynı db'deki aynı tabloları kullanıyor iseler true olarak set etmek zorundasınız.

3. DataSource tanımına bakalım;

* **#org.quartz.dataSource.myDS.jndiURL **= Comment halde duran properties, application server'da tanımlı olan jndi bilgisidir. WAS'larda commentli halde bulunur.
* **org.quartz.dataSource.myDS.jndiURL **=  Tomcat ve benzeri türevlerinde jndi'i bu sekilde alabiliyoruz.
* **org.quartz.dataSource.myDS.maxConnections** = Datasource'daki max connection bilgisidir.
* **org.quartz.dataSource.myDS.validationQuery**= Genelde connection sağlıklı mı diye verilir.


> Not : Datasource properties'lerini girerken **org.quartz.dataSource**'dan sonra **org.quartz.jobStore.dataSource** properties'ine verdiğiniz değeri girmelisiniz. Örneğimizde myDS kullandık.

Şimdi ise Job çalıştığında trigger edilecek olan class'a bakalım;

**mumin.hesap.HesapHasenâtTrigger**
``` java
package mumin.hesap;

import java.util.Date;

import org.apache.log4j.Logger;
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class HesapHasenâtTrigger implements Job {

	private static Logger logger = Logger.getLogger(HesapHasenâtTrigger.class);

	@Override
	public void execute(JobExecutionContext context)
			throws JobExecutionException {
		logger.info("**HesapHasenâtTrigger** started.");

		try {
			while (!died) {
				// do hasenat
			}
		} catch (Exception e) {
			logger.fatal(e, e);
		}

		logger.info("**HesapHasenâtTrigger** finished.");
	}

}
```

**mumin.hesap.HesapSeyyiatâtTrigger**
``` java
package mumin.hesap;

import java.util.Date;

import org.apache.log4j.Logger;
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class HesapSeyyiatâtTrigger implements Job {

	private static Logger logger = Logger.getLogger(HesapSeyyiatâtTrigger.class);

	@Override
	public void execute(JobExecutionContext context) throws JobExecutionException {
		logger.info("**HesapSeyyiatâtTrigger** started.");

		try {
			while (!died) {
				// remove seyyiât
			}
		} catch (Exception e) {
			logger.fatal(e, e);
		}

		logger.info("**HesapSeyyiatâtTrigger** finished.");
	}

}
```
Zamanı geldiğinde job trigger class'ındaki execute methodu Quartz tarafından çalıştırılacaktır. Şimdi de Cluster yaptık ama Job'lar aynı anda çalışırsa ne olacak? Örnekte sevap - günah dedik ama kritik ve önemli işlemler yapan job'ların tekrar tekrar çalışması handikap olacaktır.

Class'larımıza şunları ekleyelim mi?
<ul>
* **@DisallowConcurrentExecution** : Aynı anda aynı instance scheduler için joblar'ın çalışmasını engelle.
</ul>
Annotation class level'dir, class'ın başına koymalısınız.

Bir sonraki yazıda Java web applicaiton içerisinde quartz kullanmayı görecegiz.

Yazımı sonlandırıyorum, mutlu ve esen kalın.

> A.Akkus
