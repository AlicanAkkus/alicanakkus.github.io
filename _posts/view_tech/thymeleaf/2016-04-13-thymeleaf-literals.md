---
layout: post
title: Thymeleaf Literals
permalink: /blog/java-platform/view-tech/thymeleaf/thymeleaf-literals
summary: Bu yazımda Thymeleaf içerisinde ki literal'lere bakacağız. Öncelikle literal nedir, yenilir mi? içilir birşey mi ona bakalım.
image: images/java-platform/java-ee/jms/jms_logo.png
---
Merhaba arkadaslar,

Bu yazımda Thymeleaf içerisinde ki literal'lere bakacağız. Öncelikle literal nedir, yenilir mi? içilir birşey mi ona bakalım.

Literal, en kaba tabiriyle birşeyi ifade etme, temsil etme anlamı taşır, gösterim şekli yani. Yazılım anlamında da bu geçerlidir. Herhangi bir programlama dilinde bir değişken tanımının yapılmasından, kullanılmasına kadar birçok yerde literal mevcuttur. Belki kullanıyoruz ama farkında değiliz.

Örneğin;
``` java
int rommNo = 1;
String animalType = "cat";
```

Yukarıda integer ve string için iki literal mevcut. Öncelik **"int"**, **"String"** gibi literalde kullanılan değişkenin tipi, sonrasında değişkenin adı, assignment operatoru ile birlikte değer yer alıyor.

Literal burada tam olarak 1 degerinde oldugu gibi yada cat degerindeki "" ifadelerdir. Yani siz String tanımlamak isterseniz "" arasına karekterleri dizmelisiniz anlamındadır. Bir diğer örnek olarak mesela iki int degeri toplamak istediniz, bunun için 1 + 1 yani arada plus operatorunu kullanmanız lazım.

Literal kabaca bu anlamdadır. Asıl konumuz ise Thymeleaf içerisinde bu literalleri nasıl kullanacagımız yönünde olacaktır. Yazı içerisinde birçok literal tipini bulabilirsiniz.

Önceki yazılarda kullandıgımız projeyi tekrar kullanacagız, tabi eklemeler yaparak.

Önce ui sayfamızı besleyecek olan servlet tanımını yapalım;
``` java
package com.wora.servlet;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.log4j.Logger;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.WebContext;
import org.thymeleaf.templateresolver.ServletContextTemplateResolver;

import com.wora.bean.Order;
import com.wora.util.DateUtils;

public class LiteralsServlet extends HttpServlet {
	private static Logger logger = Logger.getLogger(LiteralsServlet.class);
	private static final long serialVersionUID = 1L;

	protected void doGet(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		execute(request, response);
	}

	protected void doPost(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		execute(request, response);
	}

	private void execute(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		logger.info("Literals page requested..");
		try {
			ServletContextTemplateResolver templateResolver = (ServletContextTemplateResolver) request
					.getServletContext().getAttribute("templateResolver");
			if (templateResolver != null) {
				TemplateEngine engine = new TemplateEngine();
				engine.setTemplateResolver(templateResolver);

				WebContext ctx = new WebContext(request, response,
						getServletConfig().getServletContext(),
						request.getLocale());
				;
				Order myOrder = new Order();
				myOrder.setId(112233);
				myOrder.setPrice(105.4);
				myOrder.setStatus(true);
				myOrder.setSummary("Phone");
				myOrder.setDetails("I bought a new phone for my sister!");

				ctx.setVariable("today", DateUtils.getLocalDate());
				ctx.setVariable("myOrder", myOrder);
				String result = engine.process("literals", ctx);

				PrintWriter out = response.getWriter();
				out.println(result);
			}
		} catch (Exception e) {
			logger.error(e, e);
		}

	}

}
```
Önceki örneklerde açıklamıştık, bir web context olusturup variable set ediyoruz. Order'ımızı literals.html sayfamızda kullanacağız. UI sayfasına bakalım;
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
</head>
<body>

<div class="container">
<div th:include="header :: header">...</div>

<div th:if="${myOrder} != null">
	<div class="panel panel-default" th:object="${myOrder}">
		<div class="panel-heading">
			<h1 class="panel-title" th:text="|Welcome to our application, your order id -> ${myOrder.id}|">Order Details</h1>
		</div>
		<div class="panel-body">
			<table class="table table-bordered">
				<thead>
					<tr>
						<th>Order ID</th>
						<th>Order Price</th>
						<th>Total Price</th>
						<th>Order Summary</th>
						<th>Order Description</th>
						<th>Status</th>
						<th th:if="${myOrder.price} > 100">Campaign</th>
					</tr>
				</thead>
				<tbody>
					<tr>
						<td th:text="${myOrder.id}">Order id</td>
						<td th:text="'$' + ${myOrder.price}">Order Price</td>
						<td th:text="'$' + ${myOrder.price + 10.0}">Total Price</td>
						<td th:text="${myOrder.summary}">Order summary</td>
						<td th:text="${myOrder.details}">Order details</td>

						<div th:switch="${myOrder.status}">
							<td th:case="true">
								<span class="label label-success" th:text="Active"></span>
							</td>
							<td th:case="false">
								<span class="label label-danger" th:text="Passive"></span>
							</td>
						</div>
						<td th:if="${myOrder.price} > 100">You win campaign!</td>
					</tr>
				</tbody>
			</table>
		</div>
		<div class="panel-footer clearfix">
			<div class="pull-right">
				<a href="#" class="btn btn-primary" th:onclick="'alert(\'' + ${myOrder.toString()} + '\');'">Learn More</a>
				<a href="#" class="btn btn-default">Go Back</a>
			</div>
		</div>
	</div>
</div>

<div th:include="footer :: footer">...</div>
</div>

</body>
</html>
```

Örnekleri açıklamadan evvel bir ss ile son halini paylasalım, order'ı ekranda gösterdik sadece;

![thymeleaf literals1](/images/view-tech/thymeleaf/thymeleaf-literals1.png)

Literal tanımlarına bakalım;
### String Literal
String literali birçok yerde kullandık, bunlardan bazıları şu şekilde;

* Normal bir şekilde String gösterimi :
	``` html <td th:text="${myOrder.id}">Order id</td> ```
* Appending String :
	``` html <td th:text="'$' + ${myOrder.price}">Order Price</td> ```
String concenation yaptık. Ücretin başına dolar işaretini ekledim
* Formatting String :
	``` html
	<td class="panel-title" th:text="|Welcome to our application, your order id -> ${myOrder.id}|">Order Details</td>
	```
Formatlamak için text'i **|** pipe karekterleri arasına alıyoruz. 


### Numeric Literal

* Comparision :
	``` html
		<th th:if="${myOrder.price} > 100">Campaign</th>
	``` 	
Ücret 100'den büyük ise ilgili blok çalışacaktır.
* Arithmetics operations :
	``` html
	<td th:text="'$' + ${myOrder.price + 10.0}">Total Price</td>
	```
Ücret'e Kdv yada Kargo ücretini ekledik :)

### Null Literal

* Normal kullanımı :
 	``` html
		<div th:if="${myOrder} != null">
	```
myOrderr degiskeni null degilse diye işlem yapmışız. == null da yapılabilir isteğe göre.

### Boolean Literal
Boolean literal olarak direk true/false üzerinde kullanabilirsiniz. Örnekte status için switch ile beraber kullandım;
``` html
<div th:switch="${myOrder.status}">
	<td th:case="true">
		<span class="label label-success" th:text="Active"></span>
	</td>
	<td th:case="false">
		<span class="label label-danger" th:text="Passive"></span>
	</td>
</div>
```

Ayrıca şu şekilde de kullanılabilir;
``` html
<td th:text="${myOrder.status}? 'Active' : 'Passive'">yes</td>
```
### Equality Literal
Numeric deger üzerinde comparators literali de kullanabiliriz;

* Price > 100 ise kampanya yapalım :
	``` html
		<td th:if="${myOrder.price} > 100">You win campaign!</td>
	```

Yazıyı burada sonlandırıyorum arkadaslar, ilgili projeyi daha önce git üzerinden paylasmıstım, son degisiklikleri de commitledim.

Proeje git üzerinden erişebilirsiniz : https://github.com/AlicanAkkus/ThymeleafTutorial

Mutlu ve esen kalın.

> A.Akkus
