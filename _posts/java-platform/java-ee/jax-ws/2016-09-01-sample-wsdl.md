---
layout: post
title: Sample WSDL
permalink: /blog/java-platform/java-ee/jax-ws/sample-wsdl
summary: Merhaba arkadaslar, jax-ws yazılarına devam ediyoruz. Bugün örnek bir wsdl oluşturup mock servisle ayağa kaldıracağız.
image: images/java-platform/java-ee/jms/jms_logo.png
---

Merhaba arkadaslar, jax-ws yazılarına devam ediyoruz. Bugün örnek bir wsdl oluşturup mock servisle ayağa kaldıracağız.

WSDL'in **Web Services Description Language** olduğunu yani web servisin nerede/nasıl çalıştığını belirten xml-based bir definition interface olduğunu önceki yazılarda ifade etmiştik.

WSDL bir kontrat olduğu için sınırlamalar, gelecek/gidecek olan mesajlar, bu mesajların tipleri, alabileceği değerler, web servisin endpoint'i vs gibi bilgileri üzerinde barındıran bir xml dosyasıdır. Bir sözleşme bir kontrattır. WSDL üzerinden client ve server anlaşır. WSDL'e dışarıdan bakan bir adam web servisin amacından bihaber olsa dahi operasyonları, input/outputları inceleyerek bir fikre sahip olabilir. Bir önceki yazımda kullandığım weather wsdl(tamamen random şekilde internetten edindim) ile soap ui'dan bir örnek gösterelim;

![soap ui 1](/images/java-platform/java-ee/jax-ws/soap-ui1.png)
![soap ui 2](/images/java-platform/java-ee/jax-ws/soap-ui2.png)

Servis üzerinde iki adet operasyon var. Biri ülke bazında diğeri ise alt kırlım olan şehir bazında. Request/Response bilgisine bakarak servisin yapısı kısmen anlaşılabilir. WSDL incelendğinde service kısmında endpoint adresini görebiliriz;

``` xml
<wsdl:service name="GlobalWeather">
       <wsdl:port name="GlobalWeatherSoap" binding="tns:GlobalWeatherSoap">
          <soap:address location="http://www.webservicex.net/globalweather.asmx"/>
       </wsdl:port>
       .....
       .....
</wsdl:service>
```

Servisin adı GlobalWeather ve endpoint olarak http://www.webservicex.net/globalweather.asmx adresini kullanıyor. Yani kısaca wsdl'ı inceleyerek hemen hemen tüm detayları görebiliyoruz.

Bir önceki yazımda portType, binding, service, types maddelerine değinmiştim. Şimdi örnek bir wsdl hazırlayalım;

**sample.wsdl**

``` xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<wsdl:definitions xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
	xmlns:tns="http://alicanakkus.com/tutorial/jaxws/sample/" xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
	xmlns:xsd="http://www.w3.org/2001/XMLSchema" name="sample"
	targetNamespace="http://alicanakkus.com/tutorial/jaxws/sample/">
	<wsdl:types>
		<xsd:schema targetNamespace="http://alicanakkus.com/tutorial/jaxws/sample/">
			<xsd:element name="sampleRequest">
				<xsd:complexType>
					<xsd:sequence>
						<xsd:element name="in" type="xsd:string" />
					</xsd:sequence>
				</xsd:complexType>
			</xsd:element>
			<xsd:element name="sampleResponse">
				<xsd:complexType>
					<xsd:sequence>
						<xsd:element name="out" type="xsd:string" />
					</xsd:sequence>
				</xsd:complexType>
			</xsd:element>
		</xsd:schema>
	</wsdl:types>
	<wsdl:message name="sampleRequest">
		<wsdl:part element="tns:sampleRequest" name="parameters" />
	</wsdl:message>
	<wsdl:message name="sampleResponse">
		<wsdl:part element="tns:sampleResponse" name="parameters" />
	</wsdl:message>
	<wsdl:portType name="sample">
		<wsdl:operation name="sampleWS">
			<wsdl:input message="tns:sampleRequest" />
			<wsdl:output message="tns:sampleResponse" />
		</wsdl:operation>
	</wsdl:portType>

	<wsdl:binding name="sampleSOAP" type="tns:sample">
		<soap:binding style="document"
			transport="http://schemas.xmlsoap.org/soap/http" />
		<wsdl:operation name="sampleWS">
			<soap:operation
				soapAction="http://alicanakkus.com/tutorial/jaxws/sample/sampleOperation" />
			<wsdl:input>
				<soap:body use="literal" />
			</wsdl:input>
			<wsdl:output>
				<soap:body use="literal" />
			</wsdl:output>
		</wsdl:operation>
	</wsdl:binding>

	<wsdl:service name="sample">
		<wsdl:port binding="tns:sampleSOAP" name="sampleSOAP"> <!-- binding -->
			<soap:address location="http://localhost:8080/jaxws/sample" /> <!-- endpoint -->
		</wsdl:port>
	</wsdl:service>

</wsdl:definitions>
```

Görüldüğü gibi root tag wsdl:definitions'dur, diğer tüm etiketleri sarmalar. Sondan başa doğru gitmeyi uygun görüyorum.

**Service**

Service etiketi operasyonlara nereden erişilebileceğini belirtir. Service içerisine port etiketi ile binding elementine referans verip bind edilen servisi ekliyoruz. Örnekte binding olarak tns:sampleSOAP binding'ine referans verdik ve bu servisin edpointi **soap:address** olarak http://localhost:8080/jaxws/sample şeklinde ayarladık.

**Binding**

Binding üzerinde operasyonları barındıran etikettir ve portType ile birbirine bağlanır. Örnekte tns:sample portType'a bağlanmış. Binding içerisinde binding style olarak document mevcut. Burada iki tip style mevcut; document ve rpc. Document style default gelen style'dır. Teoride ikisi arasında pek fark yoktur, ufak tefek detay haricinde.

**Binding - Operation**

Binding içerisinde operasyonların tanımlandığı kısımdır. <wsdl:operation olarak tanımlanır, name attribute'u bindingin referans ettiği portType içerisindeki <wsd:operation 'da yer alan name ile eşleşmelidir. Yani wsdl:binding içerisinde ki wsdl:operation'daki name ile wsdl:portType içerisinde wsdl:operation'daki name aynı olmalıdır. Uyuşmaz ise eğer portType name'i sample oldugu için şu hatayı alacağız;

**The operation specified for the 'sampleSOAP' binding is not defined for port type 'sample'. All operations specified in this binding must be defined in port type 'sample'.**

Binding içerisindeki wsdl:operation'da input/output'un ne olacağını görüyoruz. Request ve Response'da literal olacak. Yani servisimiz document literal tipinde olacaktır. İki seçenek mevcut literal ve encoded şeklinde. Literal'de xsd ile validasyon yapılabilir, encoded'da bu zordur veya xslt ile transformasyonu zordur.

Binding içerisinde örnek bir operasyon tanımladık adı sampleWS olan.

**PortType**

PortType içerisindeki operasyonda input ve output'lar artık net olarak belirtilmelidir. wsdl:input ile request ve wsdl:output ile de response belirlenmektedir.

**Messages**

İnput ve outputu sarmalayan etikettir. Messages'da elementleri işaretleyerek input ve outputta yer alacak değişkenleri belirtir.

**Request - sampleRequest**

Request olarak String tipinde "in" değişkeni beklendiği görülüyor.

**Response - sampleResponse**

Response olarak ise yine String tipinde "out" isimli bir veri dönüleceği görülüyor.

Örnek wsdl'i soap ile açalım ve mock servisi oluşturalım. Mock servisimizden response olarak bir değer dönelim.

![soap ui hello](/images/java-platform/java-ee/jax-ws/soap-ui-hello.png)

Yukarıda endpoint olarak wsdl endpoint yerine mock servisin adresini verdik. Response olarak bize out degiskeninde Hellloooo değeri dönüldü. İnput ve outputları degistirelim;

``` xml
<xsd:element name="sampleRequest">
	<xsd:complexType>
		<xsd:sequence>
			<xsd:element name="name" type="xsd:string" minOccurs="1"   />
			<xsd:element name="surname" type="xsd:string" minOccurs="1"   />
			<xsd:element name="age" type="xsd:int" />
			<xsd:element name="gender">
				<xsd:simpleType>
					<xsd:restriction base="xsd:string">
						<xsd:enumeration value="male" />
						<xsd:enumeration value="female" />
					</xsd:restriction>
				</xsd:simpleType>
			</xsd:element>
		</xsd:sequence>
	</xsd:complexType>
</xsd:element>
<xsd:element name="sampleResponse">
	<xsd:complexType>
		<xsd:sequence>
			<xsd:element name="tckn" type="xsd:long" />
		</xsd:sequence>
	</xsd:complexType>
</xsd:element>
```

Request'i name, surname, age ve gender degiskenleri alacak sekilde düzenledik. Name ve surname alanlarını minOccurs ile zorunlu hale getirdik. Client bu alanları doldurmadan servisi çağırayamacaktır. Yine gender degiskenini de sadece iki değer alacak şekilde sınırlandırdık.  Response olarak ise tckn döndürüyoruz.

![soap ui 3](/images/java-platform/java-ee/jax-ws/soap-ui3.png)

Simdi de client nasıl bizi çağıracak hep soap'dan çağıracak degiliz. WSDL'den client oluşturmak için birçok tool vs var. Core java ile yapacak olursak wsimport'u kullanmamız gereklidir. jdk altındaki bin klasöründe default olarak gelir zaten. Kullanımına bakalım;

![jax-ws wsimport](/images/java-platform/java-ee/jax-ws/jax-ws-wsimport.png)

wsimport -keep -verbose /path/of/wsdl şeklinde çalıştırdık. Birçok parametre mevcut -keep ve -verbose kullanmayı tercih ettim. -keep kullanmazsak sadece compile olak classlar oluşturulacaktır, bu parametre ile birlikte compile edilmemis java classlarını da görebilecegiz. -verbose ise wsimportun neler yaptıgını görmek istiyoruz loga yaz dedik. Home dizinime com/... ile class ve java dosyalarını olusturdu;

![jax-ws wsimport generated classes](/images/java-platform/java-ee/jax-ws/jax-ws-wsimport-class.png)

Özel olarak output folderı verebiliriz;

**wsimport -d /home/wora/blog//wsdl/ -keep -verbose /home/wora/blog/wsdl/sample.wsdl** komutunda -d ile output directory belirleyebiliriz.

Yazımı burada bitiriyorum arkadaslar, bir sonraki yazımızda örnek wsdl'den client class oluşturarak mock servise bağlanıp kod ile data çekmeye bakacağız.


Mutlu ve esen kalın.

> A.Akkus
