---
layout: post
title: Java Tips&Trciks
permalink: /blog/java-platform/core-java/javatt
summary: Java ile ilgili sıkça sorulan ,karşılaşılan hataların , ufak tüyoların ve önerilerin olacağı  , maddeler halinde bir yazı dizisi olucaktır.
---

Java ile ilgili sıkça sorulan ,karşılaşılan hataların , ufak tüyoların ve önerilerin olacağı  , maddeler halinde bir yazı dizisi olucaktır.

Sık sık eklemeler yapmaya çalışacağım..

--------------------

**17 August 2014**

**Soru** : İnterface'lerden neden obje olusturulamaz ?

**Cevab** : İnterface'ler tamamen abstract methodlara yani gövdesiz methodlara sahiptir. İçi boş olan methodun yapacagı hicbir işlevsellik yoktur. İnterface'lerin constructor'u  da yoktur. Bu nedenle new InterfaceAdi() gibi bişey yapılamaz.

**2 October 2014**

* Java resmi tanıtımında , char bir tamsayı tipi olarak geçer .Bu char'ın ; bir int , short, long ile aynı karekterde oldugunu gosterir.
* Bir koleksiyonu foreach döngüsü içerisinde kullanabilmek için o koleksiyonun Iterable interface'sini implement etmesi gerekir.

**5 October 2014**

Soru : Interface değişkenlerin değeri neden değiştirilmez ?E

Cevab : Interface variable implicit(kapalı) olarak **public static final** 'dır.Bu nedenle değişkenden çok constant(sabit)'dır.
``` java
public static final int MIN_SIZE = 4;
int MIN_SIZE = 4;
```

İki tanımlamada aynıdır ve geçerlidir.

Interface variable final olduğu için declare edildiginde değerinin de atanması gerekir.
``` java
int MIN_SIZE;
```
Yukardaki kodu çalıştırdığımızda compiler error oluşacak ve bize şu hatayı vericektir; **Java: cannot  assing a value to final variable MIN_SIZE.**

**13 October 2014**

Soru : JSP lifecycle'de bulunan **jspInit()** , **jspDestroy()** ile **_jspService()** methodlarının isimlendirilmesi neden farklıdır?

Cevap : Java da method isimlendirmede **camelCase** yapısını kullanıyoruz. Yukarıdaki methodlar java naming conventions'a uymakta. **jspService()** methodunun başındaki **underscore(_)** bize bu methodun override edilemeyeceğini belirtmektedir. Kısaca bize "don't touch" demekte :) . Aynı şekilde underscore ön eki olmayan methodları da override edebiliriz anlamı vardır.

**5 November 2014**

Soru : Java'da abstract ve final keyword'leri bir arada kullanılabilinir mi?

Cevap : Hayır, hiç bir şekilde kullanılamaz. Abstract ve final kelimeleri asla yanyana gelemez.

Kısaca abstract demek "türet" demektir yani implemente et demektir . Final ise tam tersidir. Örneğin bir methodu abstract tanımlayarak onu alt sınıflarca implemente edilmesini saglayabiliriz. Yine aynı methodu final ile tanımlarsak alt sınıflarca bu methodun yeniden tanımlanmasının önüne geçmiş oluruz.

Abstract > Türet

Final > Türetme

Bu iki kelimeninin yanyana gelememesinin dışında abstract ve private ile abstract ve static kelimeleri de yanyana gelemez. Çünkü private methoda alt sınıflar ulaşamayacak ve bizim abstract ile "türetmek" istegimiz boşa çıkmış olucaktır.
``` java
public abstract final void test();
```
Yukardaki methodu yazmaya kalkıştıgımızda kullandıgımız ide (İntellij İdea)  bize yanlış bir combinasyon kullandığımızı söyleyecektir.

<a href="http://alicanakkus.com/wp-content/uploads/2014/08/2014-11-05-030623.png"><img class="alignleft size-full wp-image-533" src="http://alicanakkus.com/wp-content/uploads/2014/08/2014-11-05-030623.png" alt="2014-11-05 03:06:23" width="413" height="60" /></a>

&nbsp;

&nbsp;

^ Aynı hata iletisini şu method tanımlamarı içinde alırız;
``` java
private abstract void test();
```
``` java
public static abstract void test();
```
**23 January 2015**

Soru : Override edilen methodlarda ne tür kurallara uymak zorundayız?

Cevap : Override edilen methodlarda şunlara dikkat edebiliriz;

* Override edilen methodlarda argüman listesi aynı olmak zorundadır.
* Argüman list aynı değilse override değil overloaded etmiş oluruz methodu!
* Return type aynı olmaldır yada subclass tipinde olabilir.
* Access modifiers olarak daha fazla kısıtlayıcı modifiers ile tanımlayamayız. Örneğin süperclass'da public olan bir method subclass içerisinde protected yapılamaz.
* Access modifiers olarak daha az kısıtlayıcı modifiers ile tanımlayabiliriz. Örneğin süperclass'da protected olan bir method subclass'da public tanımlanabilir.
* Method tanımında exception throw edilmemişse override ederken de throw edilemez.
* Süper sınıftaki method tanımında bulunan throw Exception ifadesi daha dar kapsamlı olacak şekilde yeniden declare edilebilir.
Legal ;

``` java
    //super sınıftaki method
    public void hello() throws Exception{
        System.out.println("hello-süper");
    }

    //subclassdaki method
    @Override
    public void hello() throws RuntimeException{
        System.out.println("hello-sub");
    }
```

Illegal ;

``` java
    //super sınıftaki method
    public void hello() throws RuntimeException{
        System.out.println("hello-süper");
    }

    //subclassdaki method
    @Override
    public void hello() throws Exception{
        System.out.println("hello-sub");
    }
```

* Exception kapsamını daraltabiliriz ama genişletemeyiz.
* final olan methodların override edilemeyeceğini de unutmayalım.
* Kalıtılabilen methodlar sadece override edilebilir. Bu yüzden private olan bir method override edilemez!
* Subclass'da override edilen method içerisinden super.overriddenMethodName() çağrısı yapılabilir.



**27 May 2015**

Jsp, expression içerisine yazılanlar implicit obje olan JspWriter out objesine parametre olarak gönderilir bu yüzden expression bitiminde ; kullanılmaz. Parametre olarak gönderilmesinden dolayı da void return tipte method çağrımı expression içerisinde yapılamaz. Örn;

``` java
<%!
    public void print(){
        System.out.println(new Date().toString());
    }
%>
```
Yukarıdaki methodu expression içerisinde çağırdığımızda jsp sayfamız compile olmayacaktır.
``` java
date  <%= print() %>
```
İlgili Jsp sayfası compile olmayacaktır ve hata olarak JspWriter out objesinin void tipte parametre kabul etmediğini bize bildirecektir.

**20 June 2015**

Jax-RS ile web servis yazarken kullandığımız methodlar için bazen Mime type bilgisi belirsiz yada birden fazla olabilir. Örn; ../api/books Restful url'i bize books'ları dönecektir. Burada xml, json vs gibi dönebilir. Yada ../api/book/ gibi bir url book alıp serverde save edebilir ve gelen data xml, json gibi değişik tipte datalar olabilir. Bu tip durumlarda Restful methodlar üzerinde konumlandıracağımız @Consumes ve @Produces annotation'ları önem kazanacaktır. Unsupported media type hatası almamak için bu tip durumlarda joker ifade kullanabiliriz;

* Tüm media type için @Consumes içerisinde */* ile her türlü mime type kullanılabilir.
* Return type için tüm text media type için ise @Produces içerisinde text/* yazılabilir.
* Örn; image/* , text/* , application/* gibi.

27 September 2015

Java'da iki String ifadeyi karşılaştırmak sıkça karşılaşılan bir durumdur. Bunu birden fazla çeşitle gerçekleştirebiliriz. Bunlar; == kullanmak, equals() methodu yada compareTo() methodunu kullanmak olabilir.  == ve equals() arasındaki farkı çoğu kişi biliyordur sanırım ve interview question'lar içerisinde en sık bulunan konulardandır. == ifadesi reference karşılaştırması yapar yani obje eşitliğini kontrol eder, equals() ise içerik/value karşılaştırması yapar. Bu nedenle farklı sonuçlar üretilir ikisinin kullanımında. Örn;

``` java
String literal = "abc";
String object = new String(literal);

if(literal.equals(object)){
    System.out.printf("String %s and %s are equal %n", literal, object); }
else {
    System.out.printf("String %s and %s are not equal %n", literal, object);
}

if(literal == object){
    System.out.printf("String %s and %s are same object %n", literal, object);
}else {
    System.out.printf("String %s and %s are different object %n", literal, object);
}
```
Çıktı olarak şunu görebiliriz;

**String abc and abc are equal**

**String abc and abc are different object**

String/obje compare etmek için best practice olabilecek nitelikte şunları standart hale getirmek, yazılan kodu daha sağlam yapıcaktır.

* equals() methodu obje üzerinden değil, String ifade üzerinden çağrılmalıdır. Bu sayede Null Pointer oluşmasının önüne geçilebilir. Şöyle ki;

``` java
String apple = "Apple";
String fruit = getFruit(); // methoddan fruit null gelebilir, dikkat!

// Bu kod fruit null ise exception fırlatacaktır
if(fruit.equals("Apple"){
    System.out.println("Make Apple Shake");
}
// Bu kod her zaman çalışacaktır. Fruit null olsa dahi. Bunu tercih etmeliyiz.
if("Apple".equals(fruit){
    System.out.println("Make Apple Shake");
}
```

* String yada objenin null ile compare edilmesi yine yukarıdaki gibi NullPointer sebep olabilir. Şöyle ki;

``` java
//fruit null ise exception fırlatır. Null reference üzerinden birşey çağrılamaz!
if(!fruit.equals(null)){
    System.out.printf("We have got %s today", fruit.name());
}

//yukardakine göre bunu tercih etmeliyiz. Null ise dahi calısacaktır.
if(fruit != null){
    System.out.printf("We have got %s today", fruit.name());
}
```

**17 December 2015**
Servlet, initalize olmadan önce getServletConfig() methodunun çağrılması Servlet instantiating exceptiona neden olur. Container tarafından init() methodu çağrıldıktan sonra Servlet'imiz oluşmuş olur ancak. Bu nedenle Servlet class constructor içerisinde Servlet Confige erişmek hataya neden olacaktır.  Aşağıdaki code şu hataya örnektir;

``` java
public class MyServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;

	public MyServlet() {

		final ServletConfig config = getServletConfig();
		final String email = config.getInitParameter("email");//web.xml içerisinde servlet için "email" parametresinin oldugunu varsayalim.
		getServletContext().setAttribute("email", email);

	}

	@Override
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

		PrintWriter pw = response.getWriter();
		pw.print("Email : " + getServletContext().getAttribute("email"));

		pw.close();

	}

}
```
Burada hataya neden olan kod şu kısımdır;

``` java
final ServletConfig config = getServletConfig();
```
Servlete erişmek istediğimizde aşağıdaki sonucu görebiliriz;

<img class="size-full wp-image-1062 aligncenter" src="http://alicanakkus.com/wp-content/uploads/2014/08/Screenshot-from-2015-12-17-000105.png" alt="Screenshot from 2015-12-17 00:01:05" width="849" height="276" />

&nbsp;

**17 December 2015**

Nullable pattern olarak da bilinen beklenilen durum haricinde bir olay gerçekleştiğinde exception fırlatmak yerine geçersiz/invalid değerlere sahip objeler kullanmak exceptionlara takılmamanızı ve ek null, !null kontrolleri yapmamınızı sağlar. Örn; DBden Employee getirmek istiyen bir methodunuz bulunsun.  Employee bulunmazsa hata fırlatmak yerine boş bir Employee nesnesi göndermeniz kodu global kod standartlarına uymasını ve exceptionlara takılmamanızı sağlar.
``` java
public Employee getEmployee(int id){

		if (is employee found in a db)
			return employee
		else
			return default employee instead of exception

}
```
Employee db de bulunursa dönen Employee objesinde name : Alican, surname : Akkus, id : 12345 oldugunu varsayalım. Bulunmadıgı durumlarda ise null object yeribe name : Unknow, surname : Unknow, id : Unknow gibi bir objenin gelmesini beklememiz lazım.

**20 February 2016**

Java'da String nesneleri immutable'dir, yani oluşturulduktan sonra değiştirilemez durumdadırlar. Peki String manipülasyonları nasıl yapılır ona bakalım;

Elimizde String name="çaysever" adında bir string referansı olduğunu varsayalım. Java heap'de "çaysever" için bir referans oluşturulur ve name değişkenine atanır. Name değişkeninde bir değişiklik yapalım;
``` java
1 - String name = "çaysever";
2 - System.out.println(name); // çaysever

3 - name.toUpperCase(); // name = ÇAYSEVER

4 - System.out.println(name); // çaysever

5 - name = name.toUpperCase();

6 - System.out.println(name); // ÇAYSEVER
```
3.cü satırda name değişkeninin tuttugu stringi büyük harfe çevirmekteyiz. Bu aşama da var olan string nesnesi üzerinde işlem yapılmaz, yeni bir string nesnesi üretilir ve değeri "ÇAYSEVER" olarak set edilir. Yeni nesne referansı herhangi bir degiskene atanmadıgı için 4.cü satırda tekrar ilk değeri görüyor olacağız. Aynı işlemi 5.ci satırda yapıp name değişkenine referans olarak atadıgımızda değişimi 6.ci satırda görebiliriz.

Bir de şuna bakalım;
``` java
String name = "ali";
name += " can"+ " akkus";
```
name degiskeni üzerinde yapılan işlemde jvm " can" ve " akkus" için birer string nesnesi oluşturacaktır, eklenen degerlerle birlikte toplam 3 adet string nesnesi oluşmuş olur;
1 - " can" , 2 - " akkus" , 3 - "ali can akkus". Son olarak name degiskenine olusan string nesnesinin referansı atanmaktadır.

Bir Java uygulamasında String ve char[] nesneleri memoryi %25-%40 oranında kullanmaktadır. Bilinçsizce string kullanımı OutOfMemory'e sebep olabilmekte ve uygulamanın sonlanmasına neden olmaktadır. Java kendi içerisinde string pool kullanarak bunu önlemeye yardımcı olur bir nebze. Örneğin kodun farklı yerlerinde "alican" olarak birçok String oluşturulabilir ancak Jvm tarafından heap de String pool içerisinde tek bir "alican" nesnesi bulunur ve her String s = "alican" benzeri string degiskenleri aynı referansa sahip olur. Bir diğer yöntem ise String manipülasyonu yapmak için StringBuilder ve StringBuffer nesnelerini kullanmak olacaktır. Bu iki nesne aynı olmakla beraber farkları ise StringBuffer nesnesini thread safe olmasıdır. Bu yüzden biraz yavaştır, hız açısından StringBuilder kullanılmalıdır. Bu nesneler üzerindeki string manipülasyonlarında yeni string nesneleri oluşturulmaz, var olan nesne üzerinden işlemler yapılır.

**25 Apr 2016**

String compare ederken sabit/belli olan stringin left side'da olması runtime anındaki exceptionları azaltır. Şöyle ki;
``` java
if("alican".equalsIgnoreCase(user.getName())){
    .....
}
```
"alican" stringi sabit yani belirli bir string ve sağ taraftaki user'dan alınan name degiskenini equals ediyoruz. Burada user'dan alınan name null olabilir, sorun cıkmayacaktır. Ancak kodun şu şekilde oldugunu varsayalım;
``` java
if(user.getName().equalsIgnoreCase("alican")){
    .....
}
```
Bu durumda user'dan alınan name degiskeni null ise runtime exception olacagız. Dikkat edilesi bir trick ;)

**27 May 2016**

Servlet Contaıner olarak Tomcat kullanııyor iseniz, tomcat'in default olarak session için cookie de oluşturduğu Session Id adını değiştirebilirsiniz;

JSessionID server side tarafında çalışır, client tarafından erişilemez. JSessionID'nin expired date'i de yoktur, ayrıca WebServer ve/veya Appliaction server uzun süre bir hareketlilik görmezse session cookie'yi silme insiyatifini elinde tutar.

Default olarak tomcat, sesssion için unique olan string tipinde jsessionid oluşturup set ediyor. Bunu degistirmek için tomcat conf altındaki context.xml dosyasında şu değişikliği yapalım;
``` xml
<Context sessionCookieName="JCaysever">
           Default set of monitored resources
         <WatchedResource>WEB-INF/web.xml</WatchedResource>"
</Context>
```
Örnek olarak default cookine name ile tomcat şöyle bir session id olusturur : https://webapp.com/index.jsp;jsessionid=557206C363F1267A24AB769CA0DE4529.node01

Değişiklik sonrası şunun gibi olacaktır : https://webapp.com/index.jsp;JCaysever=557206C363F1267A24AB769CA0DE4529.node01
