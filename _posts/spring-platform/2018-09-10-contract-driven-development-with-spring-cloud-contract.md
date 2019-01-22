---
layout: post
title: Contract Driven Development With Spring Cloud Contract
permalink: /blog/spring-platform/spring-cloud/contract-driven-development
summary: Consumer Driven Contract with Spring Boot Hi, in this article we will talk about the details of consumer driven development.
image: /images/spring-platform/spring-mvc/spring_mvc_controller_image.png
---

Hi, in this article we will talk about the details of consumer driven development.

#Problem
The main problem is the conflict between consumer and producer on API interface. When you are developing an API, you have to think about the comfort of your clients. If the changes that you make break the clients, it is completely a joke. In this article, we discuss some of the challenges in consumer & producer services.

#Solution
Consumer Driven Contact(CDC) ensures the correct contract between Producer and Consumer, in a distributed system or microservices for HTTP based or message based or event based software.

Spring Cloud Contract is an implementation of CDC development of JVM based language. It’s also support for the non-jvm based language. It moves TDD to the level of the software(API) design&architecture. I call it CDC -> Client Driven Development because client(consumer) drives the changes of the API of the producer.

You can find a sample application in this tutorial.

#WHY?
As I mentioned the problem and solution above, you can think why we need this approach. There are some challenges in microservice architecture design while making handshake between consumer and producer. Therefore a change made by the producer is very difficult to test on the consumer side.

When trying to test an application that communicates many other services we could do one of the things without Consumer Driven Contract as below;

Deploy all microservices with resource and perform end-to-end test.
Mock other services in tests.
Both approaches have their advantages and disadvantages.

Deploy all microservices;

Pros -> simulates production, test real services, more reliable

Cons -> long to run, hard to debug, many cost(deploy many apps, many resources such as DB, cache etc), very late feedback

Mock other services;

Pros -> very fast feedback, no need to set up the infrastructure

Cons -> not reliable, you can go to prod with passing tests and failing prod

To solve these issues Spring Cloud Contract were created.

I follow these steps while developing a new feature for application;

Prepare contract.
Generate a test from a prepared contract.
Coding with TDD and red-green style.
After the feature has been completed, you can create stubs(contract) jar.
Consumer can use created stub jar for integration between api’s.
Let’s code;

Firstly we will create a contract with Groovy DSL as below;

```groovy
import org.springframework.cloud.contract.spec.Contract

Contract.make {
    description "should retrieve account"
    request {
        method GET()
        headers {
            accept(applicationJson())
        }
        url ('/api/v1/accounts'){
            queryParameters {
                parameter("accountId", 1L)
            }
        }
    }
    response {
        status OK()
        headers {
            contentType(applicationJson())
        }
        body(file("retrieveAccountResponse.json"))
    }
}
```


Retrieve Account Contract on Producer side
We made a sample contract for retrieve account. We have requested ‘api/v1/accounts’ endpoint with ‘accountId’ query param with HTTP get method. And we sent accept header as JSON. Finally, we’ve expected that response status should be ok(200) and response header contentType should be JSON and response body should be equals to JSON response file. Response as follow;

```json
{
    "name": "Name",
    "surname": "Surname",
    "gender": "Gender",
    "gsmNumber": "GsmNumber",
    "identifier": "Identifier",
    "createdDate": 1514851199,
    "updatedDate": 1514851199
}
```

Before generating test class we should configure contract plugin;

pom.xml on producer side

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.caysever</groupId>
    <artifactId>producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>producer</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.SR1</spring-cloud.version>
        <junit-jupiter-api.version>5.2.0</junit-jupiter-api.version>
        <junit-platform-suite-api.version>1.2.0</junit-platform-suite-api.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-contract-verifier</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit-jupiter-api.version}</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit-jupiter-api.version}</version>
        </dependency>
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-suite-api</artifactId>
            <version>${junit-platform-suite-api.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-contract-maven-plugin</artifactId>
                <extensions>true</extensions>
                <configuration>
                    <!-- Provide the base class for your auto-generated tests -->
                    <baseClassForTests>com.caysever.producer.ProducerBaseContractTest</baseClassForTests>
                    <testMode>EXPLICIT</testMode>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
```

You can generate test class from contract and coding with TDD style. You can do it via generateTests maven goal as follow -> mvn org.springframework.cloud:spring-cloud-contract-maven-plugin:2.0.1.RELEASE:generateTests

Generated test placed under the target/generated-test-sources. Generated test;

```java
package com.caysever.producer;

import com.caysever.producer.ProducerBaseContractTest;
import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;
import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;
import org.junit.Test;

import static com.toomuchcoding.jsonassert.JsonAssertion.assertThatJson;
import static io.restassured.RestAssured.*;
import static org.springframework.cloud.contract.verifier.assertion.SpringCloudContractAssertions.assertThat;

public class ContractVerifierTest extends ProducerBaseContractTest {

	@Test
	public void validate_retrieveAccountContract() throws Exception {
		// given:
			RequestSpecification request = given()
					.header("Accept", "application/json");

		// when:
			Response response = given().spec(request)
					.queryParam("accountId","1")
					.get("/api/v1/accounts");

		// then:
			assertThat(response.statusCode()).isEqualTo(200);
			assertThat(response.header("Content-Type")).matches("application/json.*");
		// and:
			DocumentContext parsedJson = JsonPath.parse(response.getBody().asString());
			assertThatJson(parsedJson).field("['surname']").isEqualTo("Surname");
			assertThatJson(parsedJson).field("['updatedDate']").isEqualTo(1514851199);
			assertThatJson(parsedJson).field("['gender']").isEqualTo("Gender");
			assertThatJson(parsedJson).field("['name']").isEqualTo("Name");
			assertThatJson(parsedJson).field("['createdDate']").isEqualTo(1514851199);
			assertThatJson(parsedJson).field("['identifier']").isEqualTo("Identifier");
			assertThatJson(parsedJson).field("['gsmNumber']").isEqualTo("GsmNumber");
	}

}
```

Auto generated test from contact on producer side
You can run it as normal JUnit tests. After tests passes you can share your contract with your client&consumer.

When you modify endpoint such as rename URL or add/delete params, you should modify the contract. If you don’t modify, build could not pass.

Spring cloud contract plugin generates stub jar for you. You can deploy it to artifactory or local repo. Spring cloud contract supports different stub modes such as classpath or local m2 repo or remote artifactory. We will use local m2 mode.

Let’s consume stub jar;

```java
package com.caysever.consumer;

import com.caysever.consumer.domain.Account;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.reactive.AutoConfigureWebTestClient;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.cloud.contract.stubrunner.spring.AutoConfigureStubRunner;
import org.springframework.cloud.contract.stubrunner.spring.StubRunnerProperties;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.web.reactive.server.WebTestClient;

import static org.assertj.core.api.Assertions.assertThat;

@ExtendWith(SpringExtension.class)
@AutoConfigureWebTestClient
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureStubRunner(ids = "com.caysever:producer:+:8090", stubsMode = StubRunnerProperties.StubsMode.LOCAL)
class ProducerVerifierTest {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void should_retrieveAccountById() {
        //given
        Long accountId = 1L;

        //when
        Account account = webTestClient.get()
                .uri("http://localhost:8090/api/v1/accounts?accountId={accountId}", accountId)
                .accept(MediaType.APPLICATION_JSON)
                .exchange()
                .expectStatus().isOk()
                .expectBody(Account.class)
                .returnResult()
                .getResponseBody();

        //then
        assertThat(account).isNotNull();
        assertThat(account.getName()).isEqualTo("Name");
        assertThat(account.getSurname()).isEqualTo("Surname");
        assertThat(account.getGender()).isEqualTo("Gender");
        assertThat(account.getGsmNumber()).isEqualTo("GsmNumber");
        assertThat(account.getIdentifier()).isEqualTo("Identifier");
        assertThat(account.getCreatedDate()).isEqualTo(1514851199);
        assertThat(account.getUpdatedDate()).isEqualTo(1514851199);
    }
}
```


Contract Verifier Test on consumer side
With @AutoConfigureStubRunner annotation spring setup wiremock server easily for you. Real producer API should be up on 8090 port. And we create reel HTTP request and assert response data. If any steps fail CI/CD pipeline is not passing. Even in your local environment instead of CI server.

All source code available on [github repo](https://github.com/AlicanAkkus/spring-cloud-contract).

Thanks.