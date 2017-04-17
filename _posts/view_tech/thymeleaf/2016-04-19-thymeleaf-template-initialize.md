---
layout: post
title: Thymeleaf Template Initialize
permalink: /blog/java-platform/view-tech/thymeleaf/thymeleaf-template-initialize
summary: Merhaba arkadaşlar, bu yazımda thymeleaf template'lerimizi initalize etme ve kullanımına değinmeye çalışacağım.
image: images/java-platform/java-ee/jms/jms_logo.png
---

Merhaba arkadaslar, bu yazımda thymeleaf template'lerimizi initalize etme ve kullanımına deginmeye çalışacağım.

Önceki yazılarda belirttigim gibi Spring ile daha iyi bir ikili oluşturuyorlar ancak thymeleaf'ı tek başına da kullanabiliriz. Servlet ile kullanımını göreceğiz bugün.

### Thymeleaf Template Initialize
Öncekile bir web projesi oluşturalım ve thymeleaf için gerekli jar'ları sitesinden edinelim. Proje dizin yapısı aşağıda yer almaktadır.

![thymeleaf template initi](/images/view-tech/thymeleaf/thymeleaf-template-initialize.png)


Template initialize etmek için öncelikle Servlet listener yazacagız. Bu konuyu bilmiyorsanız blogdaki Servlet&Jsp yazılarına bakabilirsiniz.

**TemplateInitializer.java**

``` java

package com.wora.servlet;

import java.util.logging.LogManager;
import java.util.logging.Logger;

public class TemplateInitializer implements ServletContextListener {

	private Logger logger = LogManager.getLogger(TemplateInitializer.class);

	public void contextInitialized(ServletContextEvent sce) {
		logger.info("Context initializing..");

		ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(); // Resolver
		templateResolver.setTemplateMode("XHTML"); // template mode
		templateResolver.setPrefix("/WEB-INF/"); // template prefix
		templateResolver.setSuffix(".html"); // template suffix
		templateResolver.setCacheTTLMs(3600000L); // template cache time to live

		sce.getServletContext().setAttribute("templateResolver", templateResolver); // add
																					// resolver
																					// to
																					// context

		ServiceFacade.getInstance().startService();

		logger.info("Context initialized..");
	}

	public void contextDestroyed(ServletContextEvent sce) {
		logger.info("Context destroyed..");
	}
}
```

Context initialize olurken template resolver'ımızı oluşturuyoruz bu sayede daha sonra kullanacagımız template'ler için resolverımız hazır bulunuyor. Resolver oluştururken template mode olarak XHTML verdik, default olarak XHTML'dir. Farklı olarak xml, xhtml, html verilebileceginiz belirtelim. Prefix vereken WEB-INF altında ki dosyalar ile ben çalışacağım diye belirttik. Suffix ise template dosyalarımızın uzantısının .html oldugunu belirttik. Cache olarak bir zaman belirttik. Son olarak servlet context'e attribute olarak resolver'ı ekledik.

Şimdi de template dosyamız icin bir servlet olusturalım;
``` java
package com.wora.servlet;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.WebContext;
import org.thymeleaf.templateresolver.ServletContextTemplateResolver;

import com.wora.facade.ServiceFacade;

public class OrderServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		execute(request, response);
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		execute(request, response);
	}

	private void execute(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// template reolsverı alalım
		ServletContextTemplateResolver templateResolver = (ServletContextTemplateResolver) request.getServletContext().getAttribute("templateResolver");

		TemplateEngine engine = new TemplateEngine();// template engine
														// olusturlaım
		engine.setTemplateResolver(templateResolver);// template engine'e hangi
														// template'lerini
														// kullanacagını
														// resolver ile
														// bildirelim

		// webcontext ile template icin kullanılacak dataları ekleyelim
		WebContext ctx = new WebContext(request, response, getServletConfig().getServletContext(), request.getLocale());
		// webcontext bizden request response servlet context ve locale bilgisi
		// istemekte.

		try {
			// orders degiskeni olarak bir liste set ettik
			ctx.setVariable("orders", ServiceFacade.getInstance().getOrders());
		} catch (Exception e) {
			e.printStackTrace();
		}

		// engine'e işlemesi için bir template adı ve context verelim
		String result = engine.process("thy", ctx); // result

		PrintWriter out = null;
		try {
			out = response.getWriter();
			out.println(result); // response'a resultu gonderelim
		} finally {
			out.close();
		}
	}

}
```
Kod içerisine açıklamalar ekledim. Yaptıgımız is dataları bind edecegimiz bir webContext olusturmak ve buna bind etmek. Engine'e bir template ismi ile beraber webcontexti verip işledik. Son olarak result'u respınse gönderiyoruz.

Burada dikkat edilecek konu şu; "thy" ile template dosya ismi veriyoruz. Verdigimiz suffix ve prefixlerden sonra Thymeleaf, /WEB-INF altında thy.html uzantlı bir dosya arayacaktır.

Servlet'in mappingi şu şekilde;
``` xml
<servlet>
	<servlet-name>Thy</servlet-name>
	<servlet-class>com.wora.servlet.OrderServlet</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>Thy</servlet-name>
	<url-pattern>thy.html</url-pattern>
</servlet-mapping>
```
Şimde template dosyamıza bakalım, asıl konuya da gelelim;

``` html
<!DOCTYPE html SYSTEM "http://www.thymeleaf.org/dtd/xhtml1-strict-thymeleaf-4.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="X-UA-Compatible" content="IE=edge" />
<meta name="viewport"
content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<meta http-equiv="Content-Style-Type" content="text/css" />
<meta http-equiv="Cache-Control" content="no-cache" />
<meta http-equiv="Pragma" content="no-cache" />
<meta http-equiv="Expires" content="0" />
<link rel="stylesheet" th:href="@{/static/css/bootstrap.css}"></link>
<script type="text/javascript"
th:src="@{/static/js/jquery-2.2.2.min.js}"></script>
<script type="text/javascript" th:src="@{/static/js/bootstrap.min.js}"></script>
<script type="text/javascript" th:src="@{/static/js/order.js}"></script>
</head>
<body>

<div class="container">
<div th:include="header :: header">...</div>

<div class="row">
	<div class="col-sm-6">

		<form th:attr="action=@{/addOrder}" role="form" data-toggle="validator">

			<div class="form-group" id="orderIdGroup">
				<label for="orderId">Order ID:</label>
				<input type="number" max="10000000" min="1" class="form-control" name="orderId" id="orderId" placeholder="Order id.." required="required"/>
			</div>
			<div class="form-group">
				<div class="input-grouo">
					<label for="orderPrice">Order Price:</label>
					<div class="input-group-addon">$</div>
					<input type="number" step="any" max="1000" min="0.1" class="form-control" name="orderPrice" id="orderPrice" placeholder="Order price.." required="required" />
				</div>
			</div>
			<div class="form-group">
				<label for="orderSummary">Order Summary:</label>
				<input type="text" class="form-control" name="orderSummary" id="orderSummary" placeholder="Order summary.." required="required" />
			</div>
			<div class="form-group">
				<label for="orderDescription">Order Description:</label>
				<input type="text" class="form-control" id="orderDescription" name="orderDescription" placeholder="Order desc.." required="required"/>
			</div>
			<div class="radio">
				<label class="radio-inline"><input type="radio" name="orderStatus" value="true" />Active</label>
				<label class="radio-inline"><input type="radio" name="orderStatus" value="false" />Deactive</label>
			</div>
			<button id="orderFormButton" type="submit" class="btn btn-default">Save</button>

			<input id="action" name="action" type="hidden" value="save" />
		</form>
	</div>

	<div class="col-sm-6">
			<table class="table table-bordered">
				<thead>
					<tr>
						<th>#</th>
						<th>Order ID</th>
						<th>Order Price</th>
						<th>Order Summary</th>
						<th>Order Description</th>
						<th>Action</th>
					</tr>
				</thead>
				<tbody>
					<tr th:each="order, current : ${#lists.sort(orders, comparator)}" th:class="(${order.status} == true) ? 'success' : 'danger'">
						<td th:text="${current.index + 1}"></td>
						<td th:text="${order.id}"></td>
						<td th:text="${order.price} + '$'"></td>
						<td th:text="${order.summary}"></td>
						<td th:text="${order.details}"></td>
						<td>
							<div class="btn-group">
								<button type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
									Action <span class="caret"></span>
								</button>
								<ul class="dropdown-menu">
									<li><a th:href="@{/deleteOrder(orderId=${order.id})}">Delete</a></li>
									<li><a href="#" th:onclick="'javascript:edit(\'' + ${order.id} + '\' , \'' + ${order.price} + '\' , \'' + ${order.summary} + '\' , \'' + ${order.details} + '\' , \'' + ${order.status} + '\'  );'">Edit</a></li>
								</ul>
							</div>
						</td>
					</tr>
				</tbody>
			</table>
	</div>
</div>
<div th:include="footer :: footer">...</div>
</div>

</body>
</html>
```

Bir ekran görüntüsü ekleyelim ve template'i acıklayalım;

[thymeleaf template](/images/view-tech/thymeleaf/thymeleaf-template.png)

Kısaca bir order ekleme/silme/düzenleme ekranı. Bu ekranda sol tarafta order ekleme ve sağ tarafta listeleme/silme/düzenleme yapacagız.

Az önce servlet'te data olarak orders'ları **"orders"** adıyla bind etmistik. Template'de bu degiskeni nasıl kullandıgımıza bakalım;

``` html
<tr th:each="order, current : ${#lists.sort(orders, comparator)}" th:class="(${order.status} == true) ? 'success' : 'danger'">
```

Table içerisinde tr etiketi ile gelen orders'ları th:each ile itere ediyoruz. Burada order, itere edilen orders içerisindeki order'ı ifade ediyor. Current ise o anki orderın sırasını gösterebilmek için kullanacagız.

Bir önceki yazıda expression'lara deginmis, 4 adet olan expressionları anlatmıstık. Bunların nested olabileceginden bahsetmistik.

**#lists** bir **utility** objecti ve listeler konusunda bize yardımcı olacaktı. Gelen orders listesini sort etmek istedim. Sort islemini de price alanına göre yaptım. Sort edebilmek icin **#list** utility objesine listeyi ve neye göre sort edecegini bilmesi icin comparator veriyoruz.

Not : Orders'ları Java tarafından comparator yapmazsak template'de hata alacaktır.

Sort edilen listeyi variable expression olan **${...}** ile kullanacagız.

**th:class** kısmını orderın aktif olması durumunda succes, degilse danger olarak işaretkemek için kullandım. **${order.status}** alanı true/false olabilmekte, thymeleaf içerisinde conditional operatorlerin kullanımına örnek olması için özellikle ekledim.
``` html
<td th:text="${current.index + 1}"></td>
<td th:text="${order.id}"></td>
<td th:text="${order.price} + '$'"></td>
<td th:text="${order.summary}"></td>
<td th:text="${order.details}"></td>
```
Tabloyu doldururken satırlara order'ın bilgilerini girecegiz.** th:text** ile satırın degerini ayarlıyoruz. Yine variable expression kullandık. Burada asterik operator de kullanılabilir, size kalmıs bir durum.

Link expression'a örnek olarak şu kısmı ekledim;
``` html
<li><a th:href="@{/deleteOrder(orderId=${order.id})}">Delete</a></li>
```
Burada link expression ile a elementinin href degerini dinamik olarak ürettik. Link şu olacaktır: **/deleteOrder?orderId=3** gibi.

Thymeleaf içerisinde javascript function çağırmaya örnek olması için de şu kısmı ekledim;
``` html
<li><a href="#" th:onclick="'javascript:edit(\'' + ${order.id} + '\' , \'' + ${order.price} + '\' , \'' + ${order.summary} + '\' , \'' + ${order.details} + '\' , \'' + ${order.status} + '\'  );'">Edit</a></li>
```
**th:onclik** ile js dosyamızdaki **edit** fonksyionunu çağırdık ve paremetre gönderdik. Edit fonksiyonu gelen order'ı ekrandaki kutucuklara yazıp action'u add yerine update olarak degistiriyor, bu konulara girmeyecegim.

Template include etmeye bir örnek olarak da header ve footer kısımlarını ekledim;
``` html
<div th:include="header :: header">...</div>
```
İlk header paremetresi contexti ifade ediyor, yani nereye include edilecegini. İkincisi ise template'in adı.

**header.html**
``` html
<div th:fragment="header">
	<div class="jumbotron" style="background-color: #0BBBAE">
		<p align="center">
			Created By <b th:text="#{author}"></b>
		</p>
	</div>
</div>
```
**th:fragment** içerisindeki header, include ediriken yazılan ikinci paremetredir, bir nevi id gibi.

Message expressiona örnek olması için şu kısmı ekledik;
``` html
	<p align="center">
		Created By <b th:text="#{author}"></b>
	</p>
```
Message expression #{...} kullanılıyor demistik. Message properties dosyası template adı ile aynı olmalı ve **_language** olarak bitmeli. /WEB-INF altında **thy_en.properties** dosyasında author degiskeni kullanılacaktır.

**thy_en.properties**
``` java
	msg=Hello World Thymeleaf!!
	author=Alican Akkus
	created_date=10/04/2016
```
Multiple language özellikli diller yapabilmek için thy_en, thy_tr gibi properties dosyaları hazırlamanız ve template engine'de template'i işlerken Locale bilgisi olarak request'in locale bilgisini eklerseniz otomatik olarak ek birşey yapmadan çoklu dil özellikli uygulama yazmıs olacaksınız.

Yazıyı burada sonlandırıyorum arkadaslar, mutlu ve esen kalın :)

Örnek uygulamaya şuradan erişebilirsiniz; https://github.com/AlicanAkkus/ThymeleafTutorial

> A.Akkus
