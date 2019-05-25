---
layout: post
title: Dockerized Spring Boot Integration Test in Mono Repo
permalink: /blog/docker/dockerized-springboot-monorepo
summary: Hi, in this article we will talk about the dockerized integration test in the mono repo. As you know there are many different tests we write such as unit, integration, functional, feature, contract, system etc. Integration tests are one of them. Their differences are not related to this article.

---

Hi, in this article we will talk about the dockerized integration test in the mono repo.

As you know there are many different tests we write such as unit, integration, functional, feature, contract, system etc. Integration tests are one of them. Their differences are not related to this article.

At first glance, integration tests are expensive. Because of needed to set up many resources during tests. 
Sometimes you need to up the database, messaging queue, cache provider and etc. In our example, we will use the only database. 
We believe that the in-memory database is not reliable because of is not like a real system. 
We use MySQL in production but tests are running on H2 or any other in-memory databases. We can not be sure that our code runs correctly in the real system. 
Therefore we will use the real database in the test. For example **DATEADD(hour, 1, now()** is a valid syntax for **H2** but the invalid syntax for **MySQL**, correct syntax should be **DATE_ADD(now(), INTERVAL 1 HOUR)**. 
Do you still trust to the in-memory databases?

Also, we don’t want to use any tool such as TestContainer in the source code. Test source code should not create or configure infrastructure, should focus to only own test, not infra! We will use dockerized MySQL during tests.

Let’s begin;

We will use Gradle as the build tool.

In the mono repo, we have many projects/microservices. All project has an **integrationTest** task for integration tests. We’ve also **it** task for the run all of them.

When you run the integration tests, the first step is the start database in our example. Run all test after database up and running. After tests completed down the database and clean up the resource.

We’ve multiple microservice in the mono repo. All microservice creating own database/schema during the test. Each microservice independent of others.


**testing.gradle**;

```

test {
    failFast true
    testLogging {
        events "PASSED", "STARTED", "FAILED", "SKIPPED"
    }
    afterSuite { desc, result ->
        if (!desc.parent) {
            println "\nTest result: ${result.resultType}"
            println "Test summary: ${result.testCount} tests, " +
                    "${result.successfulTestCount} succeeded, " +
                    "${result.failedTestCount} failed, " +
                    "${result.skippedTestCount} skipped"
        }
    }
    useJUnitPlatform {
        includeEngines 'junit-jupiter'
        excludeTags 'integrationTest'
    }
}

task integrationTest(type: Test) {
    testLogging {
        events "PASSED", "STARTED", "FAILED", "SKIPPED"
    }
    useJUnitPlatform {
        includeTags 'integrationTest'
    }
}
```

All microservices have an ‘integrationTest’ task as mentioned above. Marks all integration test as **integrationTest** with JUnit 5 **@Tag** annotation. And we also have ‘it’ task as mentioned before.

All projects include testing.gradle file to their configuration. Add below configuration to root **build.gradle**;
```
allprojects {
 apply from: “$rootProject.projectDir/config/testing.gradle”
}
```

**dockerized-it.gradle**;
```
task dbup(type: Exec) {
    workingDir rootProject.projectDir
    commandLine 'sh', '-c', './scripts/integration-test.sh db up'
    setIgnoreExitValue(false)
}

task dbdown(type: Exec) {
    workingDir rootProject.projectDir
    commandLine 'sh', '-c', './scripts/integration-test.sh db down'
    setIgnoreExitValue(false)
}

task it() {
    doFirst {
        println 'Integration tests are going to run...'
    }

    //run db
    dependsOn dbup

    //run all integration tests
    dependsOn(allprojects.integrationTest)

    //stop db
    finalizedBy dbdown

    doLast {
        println 'Integration test are completed.'
    }

```

Let see **it** task. It’s run database and run all integration tests in the mono repo and finalized task by database down command.

Let’s look at the integration-test.sh script for up and down database. It’s very simple as below;

```
#!/usr/bin/env bash

run_db() {
    docker run --name test_db --env MYSQL_ROOT_PASSWORD=root_password -p 4306:3306 -d mysql
}

down_db() {
    docker rm -f test_db
}

case "${1}" in
    "db")
        case "${2}" in
        "up") echo Starting db && run_db;;
        "down") echo Stopping db && down_db;;
        esac;;
    *)
        echo Unknown command! ${1}
esac
```

Finally, test property file looks like below for spring boot application;
```
spring:
  application:
    name: my-api
  profiles:
    active: test

---
spring:
  profiles: test
  jpa:
    show-sql: false
    hibernate:
      ddl-auto: create-drop
    open-in-view: false
    database-platform: org.hibernate.dialect.MySQLDialect
  datasource:
    url: jdbc:mysql://localhost:4306/my-api?createDatabaseIfNotExist=true
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root_password
```

Run gradle it task via ‘gradle it’ command. The output looks like below;

```
> Task :dbup
    Starting db
    db7210b621839d667a7c2213afa773e17d5edb975dc753cde1451e057ef553c5
> Task :it
    Integration tests are going to run…
    ….some tests are running
    ….some tests are running
    Integration test are completed.
> Task :dbdown
    Stopping db
    test_db
    BUILD SUCCESSFUL in 1m 34s
    35 actionable tasks: 20 executed, 15 up-to-date
➜ mono-repo (master) ✔
```

The database is a sample for integration tests. You can create any other resources for the test are needed.

> A.Akkus