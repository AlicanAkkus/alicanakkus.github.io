---
layout: post
title: Docker Swarm Mode Introduction
permalink: /blog/docker/docker-swarm-mode-introduction-en
summary: In this post we will talk about Docker Swarm Mode.

---

In this post we will talk about Docker Swarm Mode.

We will start by talking about what is the Docker Swarm Mode. We will then look at use cases and look at the theory of work. In the next article, we will create a swarm cluster in our local machine and try it.

## Docker Swarm Mode Feature
Swarm mode is a container orchestration tool for the Docker platform. As you know, we created container with Docker and solved many problems by running our application in docker environment. Later, with the docker compose structure, we could run multiple containers with a single command.

There is a Docker Swarm to go even further. We can manage our application consisting of DB, App Server, Web Server, Cache and MQ with Docker Swarm and scale under load, solve multiple point of failure problem by running multiple instances on multiple hosts.

In the latest versions of the Docker, Swarm mode is installed in-built. Let's look at the Swarm mode capabilities;

* **Cluster management integrated with Docker Engine** : İf you know the docker you do not need to learn a separate tool for container orchestration. You can manage containers with very similar Docker CLI command set.

* **Declarative service model** : With the declarative approach, you can group and separate the components in the application stack. For example; MQ, and Redis as backend services, and Web Server components as frontend services. This may have the following advantage: By grouping services, you can also allow the frontend to prevent the grouped components from communicating directly with the backend services.

* **Scaling** : You can scale any service in the application stack at any time. You only need to specify how many instances should work for corresponding service.

* **Desired state reconciliation** : Docker Swarm continuously compares and analyzes the current state with the expected state. For example; You are working with 3 instances of your MQ service component. Expected state: 3 MQ container, Current state: 3 MQ container. There is also a container stop or inaccessible, in which case the current state will be 2 containers and will be incompatible with the expected state. Docker Swarm will detect this and immediately the 3rd container will be created for you.

* **Service discovery** : One of the features that come in-built in Docker Swarm is the embedded DNS server. Let's move on through our example again; If your APP Server component wishes to access the MQ service in the Swarm environment, only "MQ" is enough, you do not need ip information. Docker Swarm will create a unique DNS record for each service.

* **Load balancing** : Another feature that comes in-built in Docker Swarm is load balancing. If there are 4 Web Server instances, Docker Swarm intelligently balances incoming requests. If you wish, you can connect your services to a different load balancer (Netscaler, F5).

* **Secure by default** : Each host in the Docker Swarm mode environment communicates with TLS. You can give your certificate to the Docker, by default Swarm will create one for you.

* **Rolling updates** : Assume that you have 3 instances running Redis 3.0.6, and you will upgrade the version. Docker Swarm will give you this easily. Redis 3.0.7 will be upgraded to each instance. In case of any failures, the existing structure will be converted by rollback.

> Note: Docker Swarm keeps the previous version of each service. You can also roll back manually at any time.

### Docker Swarm Mode Use Cases

* With Docker Swarm mode, you can manage containers like we mentioned above, use multiple hosts as a single host, scale and use them.

* It makes development easier. He solved the biggest problems of developers ** he was working on my machine. For example; In the Prod environment there is a load balancer at the front, with more than one node at the back. One problem is that the developer can not solve the problem completely because of developer could not build environment as production at the local machine. Environment will be similar to each other with Docker Swarm.
Briefly; Your application will work the same in all environments.

### Docker Swarm Mode

We have stated that the Docker Swarm has more than one host (node). Nodes perform different tasks according to the role they have. This roles;

1. **Manager** : Manager node have the ability to manage the docker swarm, and they can read and modify swarm metadata.

2. **Worker** : The tasks in the worker role do the job. Manager tells which service will work, and the worker also runs it.

> Note: By default, Manager nodes can run any tasks, so they can run services. But usually is not the right approach. You can ask why you are doing the default job :) My answer is: In order to work with the Docker Swarm mode, you must have at least one manager node in swarm. There is no requirement to worker. So you want to try SWARM and you can only try it by creating a manager node. You do not need to set up a second node. If we do not want the manager node run task, we will switch this feature.

Let's look at the Docker Swarm Mode architecture;

![docker swarm mode](/images/docker/docker-swarm-mode.png)

We will see that the managers communicate with each other and with the workers when they look at the architecture. Workers do not communicate with each other.

Manager nodes use the RAFT algorithm to provide consensus among themselves. For example; leader election, RAFT is used in Docker Swarm. We will not go into much detail, but ** RAFT ** tells us that your cluster environment will work well with this math.

**(N - 1) / 2** : For example, if there are 3 manager nodes in your cluster environment (3-1) / 2 = 1, ie 1 manager node down, your cluster architecture will not be affected.

> Note: Increasing Manager node number will not improve performance. As the number of manager nodes increases, it will be long and difficult to obtain consensus. In general, the recommended maximum number of manager nodes is 7.

We have pronounced the service word many times. Services in Docker Swarm Mode also appear as tasks. The above mentioned components MQ, Redis, etc are some of the services. If they are working as a container, it is a task. Let's try to show it right below;

![docker swarm mode service](/images/docker/docker-service.png)

Bir nginx servis tanımımız var ve biz bundan 3 tane instance çalıştırılmasını istemişiz. **nginx** burada ki servisimiz, node'lar üzerinde çalışan **nginx.1**, **nginx.2**, **nginx.3** ise tasklarımız olmaktadır. Bu tasklardan herhangi birinin göçmesi durumunda Docker Swarm tarafından yeni task oluşturulup çalıştırılacaktır.

We have an nginx service definition and we want to run 3 instances of it. ** nginx ** is the our service, **nginx.1**, **nginx.2**, **nginx.3** are tasks working on nodes. Docker Swarm will create and run a new task if any of these tasks is down.

> Note: Many times we talked about instance number like 2, 3 and 4. It is recommended that the best practice is to set the number of instances to 0 for relevant service instead of deleting / removing a service.

#### Docker Swarm Mode Service Type

The services we mentioned are only working in the swarm environment. So if you do not start a swarm or join swarm, you can not create a services on the docker engine. In Swarm environment, service will create only in the swarm manager node. In the next article we will make a lot of examples about services but it is beneficial to have theoretical knowledge.

There are 2 types of services. Global type and Replicated type.

* **Global service** : Global services are running in all the nodes (manager & worker) in the swarm environment and the number of instances can not be given. This service is also run by default for each newly added node. You can define a global type for your services, such as monitoring agents, anti-virus scanners, that you want to run on each node.

* **Replicated service** : Replicated service is the type you decide how many instance will work.

Choosing one of the two types is quite easy, you can set it with the **--mode** parameter when creating a service. We will make an example in the next article.

Let's look at the image;

![docker swarm mode service](/images/docker/docker-replica-and-global-services.png)

I end up writing Docker Swarm Mode here, next time we will create our own swarm environment and launch services.

If you think there is any mistake in the article, you can contribute [here](https://github.com/AlicanAkkus/alicanakkus.github.io).

> Alican Akkus.
