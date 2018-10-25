- [language or framework design patterns](#sec-1)
- [general architecture](#sec-2)
  - [10 common architectural patterns](#sec-2-1)
    - [layered pattern](#sec-2-1-1)
    - [client-server pattern](#sec-2-1-2)
    - [master-slave pattern](#sec-2-1-3)
    - [pipe-filter pattern](#sec-2-1-4)
    - [broker pattern](#sec-2-1-5)
    - [peer-to-peer pattern](#sec-2-1-6)
    - [event-bus pattern](#sec-2-1-7)
    - [model-view-controller pattern](#sec-2-1-8)
    - [blackboard pattern](#sec-2-1-9)
    - [interpreter pattern](#sec-2-1-10)
  - [reactive design patterns](#sec-2-2)
    - [fault tolerance and recovery patterns](#sec-2-2-1)
    - [replication patterns](#sec-2-2-2)
    - [resource management patterns](#sec-2-2-3)
    - [message flow patterns](#sec-2-2-4)
    - [flow control patterns](#sec-2-2-5)
    - [state management and persistence patterns](#sec-2-2-6)
  - [scalable system design patterns](#sec-2-3)
    - [load balancer](#sec-2-3-1)
    - [scatter and gather](#sec-2-3-2)
    - [result cache](#sec-2-3-3)
    - [shared space](#sec-2-3-4)
    - [pipe and filter](#sec-2-3-5)
    - [map reduce](#sec-2-3-6)
    - [bulk synchronous parallel](#sec-2-3-7)
    - [execution orchestrator](#sec-2-3-8)
  - [patterns of enterprise application architecture](#sec-2-4)
    - [domain logic patterns](#sec-2-4-1)
    - [data source architectural patterns](#sec-2-4-2)
    - [object-relational behavioral patterns](#sec-2-4-3)
    - [object-relational structural patterns](#sec-2-4-4)
    - [object-relational metadata mapping patterns](#sec-2-4-5)
    - [web presentation patterns](#sec-2-4-6)
    - [distribution patterns](#sec-2-4-7)
    - [offline concurrency patterns](#sec-2-4-8)
    - [session state patterns](#sec-2-4-9)
    - [base patterns](#sec-2-4-10)
  - [system design primer](#sec-2-5)
    - [review the scalability video lecture and article](#sec-2-5-1)
    - [performance vs scalability](#sec-2-5-2)
    - [latency vs throughput](#sec-2-5-3)
    - [availability vs consistency](#sec-2-5-4)
    - [consistency patterns](#sec-2-5-5)
    - [availability patterns](#sec-2-5-6)
    - [domain name system](#sec-2-5-7)
    - [content delivery network](#sec-2-5-8)
    - [load balancer](#sec-2-5-9)
    - [reverse proxy (web server)](#sec-2-5-10)
    - [application layer](#sec-2-5-11)
    - [database](#sec-2-5-12)
    - [cache](#sec-2-5-13)
    - [asynchronism](#sec-2-5-14)
    - [communication](#sec-2-5-15)
    - [security](#sec-2-5-16)
    - [appendix](#sec-2-5-17)
    - [system design interview](#sec-2-5-18)
    - [object-oriented design interview](#sec-2-5-19)
  - [architecting for reliability](#sec-2-6)
    - [concepts](#sec-2-6-1)
    - [resiliency and availability design patterns for the cloud](#sec-2-6-2)
    - [high availability architectures](#sec-2-6-3)
- [cloud architecture](#sec-3)
  - [aws cloud design patterns](#sec-3-1)
  - [azure cloud design patterns](#sec-3-2)
  - [cloud patterns](#sec-3-3)
  - [cloud computing patterns](#sec-3-4)
  - [google cloud solutions](#sec-3-5)
- [serverless architecture](#sec-4)
  - [serverless architecture five design patterns](#sec-4-1)
  - [solving problems in serverless](#sec-4-2)
- [micro services and distributed systems](#sec-5)
  - [microservice patterns](#sec-5-1)
  - [microservices](#sec-5-2)
  - [microservices anti patterns](#sec-5-3)
  - [12 factor](#sec-5-4)
  - [microservices sync vs async](#sec-5-5)
  - [message queues](#sec-5-6)
  - [enterprise integration patterns](#sec-5-7)
- [internet of things](#sec-6)
  - [iot communication patterns](#sec-6-1)
  - [design patterns for iot](#sec-6-2)
- [big data](#sec-7)
  - [big data patterns](#sec-7-1)
  - [map reduce patterns](#sec-7-2)
  - [streaming realtime analytics](#sec-7-3)
- [databases](#sec-8)
  - [sql](#sec-8-1)
    - [database tenancy patterns](#sec-8-1-1)
    - [database answers](#sec-8-1-2)
    - [database programmer](#sec-8-1-3)
    - [red gate](#sec-8-1-4)
    - [sqlcheck](#sec-8-1-5)
  - [nosql](#sec-8-2)
    - [nosql resilience patterns](#sec-8-2-1)
    - [nosql patterns](#sec-8-2-2)
    - [mongodb](#sec-8-2-3)
- [docker and devops](#sec-9)
  - [containers patterns](#sec-9-1)
  - [container anti patterns](#sec-9-2)
  - [kubernetes](#sec-9-3)
  - [container design patterns](#sec-9-4)
  - [pattern and anti pattern cicd](#sec-9-5)
  - [best practices for shell scripts](#sec-9-6)
- [mobile](#sec-10)
  - [ios](#sec-10-1)
    - [ios architecture patterns](#sec-10-1-1)
  - [android](#sec-10-2)
    - [android patterns](#sec-10-2-1)
    - [design patterns for android](#sec-10-2-2)
    - [mvc mvp and mvvm](#sec-10-2-3)
- [front end development](#sec-11)
  - [user interface](#sec-11-1)
  - [oocss acss bem smacss](#sec-11-2)
  - [css pro tips](#sec-11-3)
  - [responsive design patterns](#sec-11-4)
  - [front end architecture](#sec-11-5)
    - [mv\*](#sec-11-5-1)
    - [gui architectures](#sec-11-5-2)
- [security](#sec-12)
  - [open security architecture](#sec-12-1)
  - [web security basics](#sec-12-2)
  - [cloud security](#sec-12-3)
  - [owasp](#sec-12-4)
  - [azure security](#sec-12-5)
- [books](#sec-13)
  - [django design patterns and best practices](#sec-13-1)
  - [mongodb applied design patterns](#sec-13-2)
  - [design patterns elements reusable object oriented](#sec-13-3)
  - [head first design patterns brain friendly](#sec-13-4)
  - [effective java](#sec-13-5)
  - [node js design patterns](#sec-13-6)
  - [game programming patterns](#sec-13-7)

# language or framework design patterns<a id="sec-1"></a>

# general architecture<a id="sec-2"></a>

## 10 common architectural patterns<a id="sec-2-1"></a>

### layered pattern<a id="sec-2-1-1"></a>

### client-server pattern<a id="sec-2-1-2"></a>

### master-slave pattern<a id="sec-2-1-3"></a>

### pipe-filter pattern<a id="sec-2-1-4"></a>

### broker pattern<a id="sec-2-1-5"></a>

### peer-to-peer pattern<a id="sec-2-1-6"></a>

### event-bus pattern<a id="sec-2-1-7"></a>

### model-view-controller pattern<a id="sec-2-1-8"></a>

### blackboard pattern<a id="sec-2-1-9"></a>

### interpreter pattern<a id="sec-2-1-10"></a>

## reactive design patterns<a id="sec-2-2"></a>

### fault tolerance and recovery patterns<a id="sec-2-2-1"></a>

1.  simple component

2.  error kernel

3.  let-it-crash

4.  circuit breaker

### replication patterns<a id="sec-2-2-2"></a>

1.  active-passive

2.  multiple-master

    1.  consensus-based

    2.  conflict detection and resolution

    3.  conflict-free replicated data types

3.  active-active

### resource management patterns<a id="sec-2-2-3"></a>

1.  resource encapsulation

2.  resource loan

3.  complex command

4.  resource pool

5.  managed blocking

### message flow patterns<a id="sec-2-2-4"></a>

1.  request-response

2.  self-contained message

3.  ask pattern

4.  forward flow

5.  aggregator

6.  saga

7.  business handshake

### flow control patterns<a id="sec-2-2-5"></a>

1.  pull

2.  managed queue

3.  drop

4.  throttling

### state management and persistence patterns<a id="sec-2-2-6"></a>

1.  domain object

2.  sharding

3.  event sourcing

4.  event stream

## scalable system design patterns<a id="sec-2-3"></a>

### load balancer<a id="sec-2-3-1"></a>

### scatter and gather<a id="sec-2-3-2"></a>

### result cache<a id="sec-2-3-3"></a>

### shared space<a id="sec-2-3-4"></a>

### pipe and filter<a id="sec-2-3-5"></a>

### map reduce<a id="sec-2-3-6"></a>

### bulk synchronous parallel<a id="sec-2-3-7"></a>

### execution orchestrator<a id="sec-2-3-8"></a>

## patterns of enterprise application architecture<a id="sec-2-4"></a>

### domain logic patterns<a id="sec-2-4-1"></a>

1.  transaction script

2.  domain model

3.  table module

4.  service layer

### data source architectural patterns<a id="sec-2-4-2"></a>

1.  table data gateway

2.  row data gateway

3.  active record

4.  data mapper

### object-relational behavioral patterns<a id="sec-2-4-3"></a>

1.  unit of work

2.  identity map

3.  lazy load

### object-relational structural patterns<a id="sec-2-4-4"></a>

1.  identity field

2.  foreign key mapping

3.  association table mapping

4.  dependent mapping

5.  embedded value

6.  serialized lob

7.  single table inheritance

8.  class table inheritance

9.  concrete table inheritance

10. inheritance mappers

### object-relational metadata mapping patterns<a id="sec-2-4-5"></a>

1.  metadata mapping

2.  query object

3.  repository

### web presentation patterns<a id="sec-2-4-6"></a>

1.  model view controller

2.  page controller

3.  front controller

4.  template view

5.  transform view

6.  two-step view

7.  application controller

### distribution patterns<a id="sec-2-4-7"></a>

1.  remote facade

2.  data transfer object

### offline concurrency patterns<a id="sec-2-4-8"></a>

1.  optimistic offline lock

2.  pessimistic offline lock

3.  coarse grained lock

4.  implicit lock

### session state patterns<a id="sec-2-4-9"></a>

1.  client session state

2.  server session state

3.  database session state

### base patterns<a id="sec-2-4-10"></a>

1.  gateway

2.  mapper

3.  layer supertype

4.  separated interface

5.  registry

6.  value object

7.  money

8.  special case

9.  plugin

10. service stub

11. record set

## system design primer<a id="sec-2-5"></a>

### review the scalability video lecture and article<a id="sec-2-5-1"></a>

### performance vs scalability<a id="sec-2-5-2"></a>

### latency vs throughput<a id="sec-2-5-3"></a>

### availability vs consistency<a id="sec-2-5-4"></a>

1.  cap theorem

    1.  cp - consistency and partition tolerance

    2.  ap - availability and partition tolerance

### consistency patterns<a id="sec-2-5-5"></a>

1.  weak consistency

2.  eventual consistency

3.  strong consistency

### availability patterns<a id="sec-2-5-6"></a>

1.  fail-over

2.  replication

### domain name system<a id="sec-2-5-7"></a>

### content delivery network<a id="sec-2-5-8"></a>

1.  push cdns

2.  pull cdns

### load balancer<a id="sec-2-5-9"></a>

1.  active-passive

2.  active-active

3.  layer 4 load balancing

4.  layer 7 load balancing

5.  horizontal scaling

### reverse proxy (web server)<a id="sec-2-5-10"></a>

1.  load balancer vs reverse proxy

### application layer<a id="sec-2-5-11"></a>

1.  microservices

2.  service discovery

### database<a id="sec-2-5-12"></a>

1.  relational database management system (rdbms)

    1.  master-slave replication

    2.  master-master replication

    3.  federation

    4.  sharding

    5.  denormalization

    6.  sql tuning

2.  nosql

    1.  key-value store

    2.  document store

    3.  wide column store

    4.  graph database

3.  sql or nosql

### cache<a id="sec-2-5-13"></a>

1.  client caching

2.  cdn caching

3.  web server caching

4.  database caching

5.  application caching

6.  caching at the database query level

7.  caching at the object level

8.  when to update the cache

    1.  cache-aside

    2.  write-through

    3.  write-behind (write-back)

    4.  refresh-ahead

### asynchronism<a id="sec-2-5-14"></a>

1.  message queues

2.  task queues

3.  back pressure

### communication<a id="sec-2-5-15"></a>

1.  transmission control protocol (tcp)

2.  user datagram protocol (udp)

3.  remote procedure call (rpc)

4.  representational state transfer (rest)

### security<a id="sec-2-5-16"></a>

### appendix<a id="sec-2-5-17"></a>

1.  powers of two table

2.  latency numbers every programmer should know

3.  additional system design interview questions

4.  real world architectures

5.  company architectures

6.  company engineering blogs

### system design interview<a id="sec-2-5-18"></a>

### object-oriented design interview<a id="sec-2-5-19"></a>

## architecting for reliability<a id="sec-2-6"></a>

### concepts<a id="sec-2-6-1"></a>

1.  reliability

2.  availability

3.  resilient network design - key guidelines

4.  application design for high availability

    1.  understanding availability needs

    2.  application design for availability

5.  operational considerations for availability

    1.  automate deployments

    2.  testing

    3.  monitoring and alerting

    4.  generation

    5.  aggregation

    6.  real-time processing and alarming

    7.  storage and analytics

    8.  operational readiness reviews (orrs)

    9.  auditing

### resiliency and availability design patterns for the cloud<a id="sec-2-6-2"></a>

1.  availability patterns

2.  health endpoint monitoring

3.  queue-based load leveling

4.  throttling

5.  resiliency patterns

6.  bulk head

7.  circuit breaker

8.  compensating transaction

9.  leader election

10. retry

11. scheduler agent supervisor

12. aws specific patterns

13. multi-server pattern

14. multi-datacenter pattern

15. floating ip pattern

### high availability architectures<a id="sec-2-6-3"></a>

1.  availability goal scenarios

2.  99% scenario

3.  99.9% scenario

4.  99.99% scenario

5.  multi-region deployments

6.  99.95% scenario using multi-region deployment

7.  99.999% or higher scenario

# cloud architecture<a id="sec-3"></a>

## aws cloud design patterns<a id="sec-3-1"></a>

## azure cloud design patterns<a id="sec-3-2"></a>

## cloud patterns<a id="sec-3-3"></a>

## cloud computing patterns<a id="sec-3-4"></a>

## google cloud solutions<a id="sec-3-5"></a>

# serverless architecture<a id="sec-4"></a>

## serverless architecture five design patterns<a id="sec-4-1"></a>

## solving problems in serverless<a id="sec-4-2"></a>

# micro services and distributed systems<a id="sec-5"></a>

## microservice patterns<a id="sec-5-1"></a>

## microservices<a id="sec-5-2"></a>

## microservices anti patterns<a id="sec-5-3"></a>

## 12 factor<a id="sec-5-4"></a>

## microservices sync vs async<a id="sec-5-5"></a>

## message queues<a id="sec-5-6"></a>

## enterprise integration patterns<a id="sec-5-7"></a>

# internet of things<a id="sec-6"></a>

## iot communication patterns<a id="sec-6-1"></a>

## design patterns for iot<a id="sec-6-2"></a>

# big data<a id="sec-7"></a>

## big data patterns<a id="sec-7-1"></a>

## map reduce patterns<a id="sec-7-2"></a>

## streaming realtime analytics<a id="sec-7-3"></a>

# databases<a id="sec-8"></a>

## sql<a id="sec-8-1"></a>

### database tenancy patterns<a id="sec-8-1-1"></a>

### database answers<a id="sec-8-1-2"></a>

### database programmer<a id="sec-8-1-3"></a>

### red gate<a id="sec-8-1-4"></a>

### sqlcheck<a id="sec-8-1-5"></a>

## nosql<a id="sec-8-2"></a>

### nosql resilience patterns<a id="sec-8-2-1"></a>

### nosql patterns<a id="sec-8-2-2"></a>

### mongodb<a id="sec-8-2-3"></a>

# docker and devops<a id="sec-9"></a>

## containers patterns<a id="sec-9-1"></a>

## container anti patterns<a id="sec-9-2"></a>

## kubernetes<a id="sec-9-3"></a>

## container design patterns<a id="sec-9-4"></a>

## pattern and anti pattern cicd<a id="sec-9-5"></a>

## best practices for shell scripts<a id="sec-9-6"></a>

# mobile<a id="sec-10"></a>

## ios<a id="sec-10-1"></a>

### ios architecture patterns<a id="sec-10-1-1"></a>

## android<a id="sec-10-2"></a>

### android patterns<a id="sec-10-2-1"></a>

### design patterns for android<a id="sec-10-2-2"></a>

### mvc mvp and mvvm<a id="sec-10-2-3"></a>

# front end development<a id="sec-11"></a>

## user interface<a id="sec-11-1"></a>

## oocss acss bem smacss<a id="sec-11-2"></a>

## css pro tips<a id="sec-11-3"></a>

## responsive design patterns<a id="sec-11-4"></a>

## front end architecture<a id="sec-11-5"></a>

### mv\*<a id="sec-11-5-1"></a>

### gui architectures<a id="sec-11-5-2"></a>

# security<a id="sec-12"></a>

## open security architecture<a id="sec-12-1"></a>

## web security basics<a id="sec-12-2"></a>

## cloud security<a id="sec-12-3"></a>

## owasp<a id="sec-12-4"></a>

## azure security<a id="sec-12-5"></a>

# books<a id="sec-13"></a>

## django design patterns and best practices<a id="sec-13-1"></a>

## mongodb applied design patterns<a id="sec-13-2"></a>

## design patterns elements reusable object oriented<a id="sec-13-3"></a>

## head first design patterns brain friendly<a id="sec-13-4"></a>

## effective java<a id="sec-13-5"></a>

## node js design patterns<a id="sec-13-6"></a>

## game programming patterns<a id="sec-13-7"></a>
