---
layout: post
title:  "知识连线：从集成测试到Linux信号"
description: "记录CI过程及问题"
tags: [CI, Docker, Linux]
categories: 技术
---

# 知识连线：从集成测试到Linux信号

###引子
在软件研发领域，CI/CD从来都是人们话题。在我个人接触软件开发的初始(六/七年前)，CI/CD还只是运维人员的事，单纯的开发人员很少接触。但随着DevOps理念，容器技术及微服务架构的大行其道，CI/CD任务从开发整体过程中迁移，逐渐变成了开发人员的本职工作。CI/CD是个很大的话题，我们不展开，本篇文章的讨论范围，仅限于本人实践中与CI、Linux基础相关的点滴知识。

### 单元测试与集成测试
在软件开发过程中，开发人员首先需要面对的是什么？我认为是测试(抛开需求/设计等等)，合理与完善的测试是软件质量的基础保障。无论是TDD/BDD/DDD等等哪种开发模式，单元测试与集成测试都是绕不开的，那么两者的区别是什么呢？
* 首先，侧重点不同：
  * 单元测试侧重代码逻辑/具体功能、算法的正确性，往往是非常深入细节，外部依赖(如数据库访问/交互等)不是单元测试的重点；
  * 而集成测试侧重功能模块之间的联动，强调程序组件接口调用的正确性；
* 然后，由于侧重点不同，测试的具体方法也就不一样：
  * 在单元测试中，我们往往把代码逻辑的外部依赖(输入输出/数据库查询/RPC调用等)按照预先设计好的接口进行Mock，提供固定格式的测试数据，专注于具体代码逻辑及算法，编写各种测试用例，以求尽量完整覆盖不同逻辑分支(if...else)/不同输入条件；
  * 而集成测试时，一般把各个组件按照设计好的调用关系组合起来，从整体的功能出发做完整流程的测试，通常不再使用Mock数据，而是使用真实的输入输出和中间件进行测试。负责串联起整个流程的，有时候是对外暴漏的API，有时候直接就是UI，需要Mock的就是API的输入参数或者用户界面操作。

### 具体怎么做呢？
对于单元测试，我们需要掌握各种Mock技术：Mock外部函数，Mock输入数据，以及Mock Libiray，总之Mock是门艺术，不要轻视。

集成测试呢？在容器技术流行之前，集成测试是困难的：需要搞定各种中间件(数据库/消息队列/缓存)，还有各种上下游系统，往往需要开发/运维等等人员参与，多部门联动也是常事。但有了容器之后，只要搞到各个组件的镜像，开发人员只需要定义依赖关系，就可以搞定整个集成环境。当然，需要我们熟练掌握容器的使用、shell等多种自动化技术。

比如一套典型的Flask + Celery实现的系统，其组件大概包括：
* web-API
* DB
* broker(rabbitMQ/Radis)
* celery-worker

那么我们可以这样组织容器(docker compose)：
* 中间件服务：
<pre><code>services:
    mysql:
      image: mysql
      ...
    rabbitmq:
      image: rabbitmq
      ...
</code></pre>
* 组件服务:
<pre><code>service:
     web_api:
        image: xxxxx
     worker:
        image: xxxxxx
</code></pre>
只要搞定了服务镜像，然后需要做的就是针对分组做相应的配置及初始化，那么集成环境基本搞定。

那么接下来就开始集成测试，Python下当然还是使用unittest模块，组织测试用例，使用Mock数据，模拟用户登陆->资源创建->资源操作->资源删除等等一系列API动作，组件间合作的怎么样就很清楚了。

# 测试覆盖率
那么集成测试中的代码覆盖率怎么获取呢？

Python下我们当然用coverage(https://pypi.org/project/coverage/)，我们可以利用coverage的run命令在各个容器中启动相应服务，在集成测试跑完后收集各个服务生成的coverage文件合并(combine)为总体的覆盖率文件。

理论上看没什么问题，但在具体实践时还确实让我困扰了一番，下面具体介绍这个困扰，同时进入文章的另一个主题：Linux系统信号。

我们具体实现集成测试获取覆盖率：
* 启动服务：
<pre>docker exec -id web_container_id bash -c "coverage run -m web_api"  # -d让任务后台执行，不阻塞后续命令
docker exec -id worker_container_id bash -c "coverage run -m worker"
</pre>
* 然后我们跑测试用例：
<pre>docker exec -i web_container_id bash -c "coverage run -m test"</pre>
测试用例跑完后会自动停止，然后生成相应的coverage文件，但是服务进程一直在后台运行，该怎么停止服务然后获得coverage文件呢？

我一开始想到使用docker restart或者docker stop，连同整个container将服务进程杀掉，但是实际情况这样无法生成coverage文件；
没问题，我们还有别的办法：使用杀进程的方式：
* 首先获取服务进程ID:
<pre>web_server_id = 'docker exec -i web_container_id bash -c "ps -ef |grep web_api | awk '{print $2}'"'
worker_server_id = 'docker exec -i worker_container_id bash -c "ps -ef |grep worker | awk '{print $2}'"'</pre>

* 然后我们杀进程：
<pre>docker exec -i web_server_id bash -c "kill ${web_server_id}"</pre>
结果同样很悲剧，没有coverage文件生成。那该怎么办呢？

回过头来想：使用control + C杀掉前台进程是可以获得coverage文件的，那么怎么模拟control + C呢？

其实很简单，使用kill -SIGINT PID即可模拟control+C(本人愚钝，在同事指导下才了解这种操作)。

其实不仅是control + C这种键盘操作的退出，kill还支持很多类型的信号，可以用kill -l 来查看，常用的kill -9 也是其中一种。

在Linux系统中，信号是一种重要的进程间通信机制，用途很广泛，相关的知识点就不再展开。在我们自己的某些应用程序中，可以利用signal(https://docs.python.org/2/library/signal.html)库来处理信号，这里需要注意的是：系统中的SIGKILL和SIGSTOP信号是无法被应用程序捕获的。

# 总结
以上梳理了单元测试和集成测试，介绍了一些利用容器进行CI的思路，以及一些实践过程中的小细节，希望对读者有所帮助；

涉及的知识点不少，不过都是一带而过，今后有机会逐个深入探讨。