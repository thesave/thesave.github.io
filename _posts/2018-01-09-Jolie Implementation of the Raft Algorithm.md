---
layout: post
title:  "A Microservice Implementation of the Raft Consensus Protocol"
date:   2018-01-09 10:00:00
categories: jolie raft
---

[Raft](https://raft.github.io/) is a distributed consensus algorithm: it offers a generic way to distribute a state machine across a cluster of computing systems, ensuring that each node in the cluster agrees upon the same series of state transitions. 

The main motivation that drove Ongaro and Ousterhout in the design of Raft was to make it easy to understand and reason about.

Last year (2017) I empirically tested Ongaro and Ousterhout's thesis by giving [as laboratory assignment](http://cs.unibo.it/~sgiallor/teaching/project/current/project.pdf) for the second-year bachelor course on [Operating Systems](http://www.unibo.it/en/teaching/course-unit-catalogue/course-unit/2017/320661) ([Information Science for Management](http://www.unibo.it/en/teaching/degree-programmes/programme/2017/8014), [DISI](http://www.cse.unibo.it/en), [UniBo](http://www.unibo.it/en)) the implementation of an online shop, backed by a cluster of servers running the Raft protocol.

For the delivery, I strongly advised the students to use [Jolie](http://www.jolie-lang.org/) as programming language. 

Why Jolie? 

<img style="float:right; width:50%; margin-left:5%;margin-bottom:1%;" src="http://thesave.github.io/imgs/jolie_raft.png" alt="">
[Previous successful experiences](http://cs.unibo.it/~sgiallor/teaching#projects). For one, with Jolie students implicitly become familiar with the popular architectural style of [microservices](https://en.wikipedia.org/wiki/Microservices). Second, by developing with Jolie, neophyte students can rapidly grasp the essential concepts behind distributed applications and create complex systems. Third, Jolie lifts a lot of the complexity of setting up and running a distributed architecture, which lets programmers focus on getting their distributed logic right, rather than spending time and time on network bindings, data marshalling and transmission, etc.

Among the many excellent implementations [that I received](http://cs.unibo.it/~sgiallor/teaching/project/current/groups.html), the one delivered by Matteo Berti, Arnaldo Cesco, and Salvatore Fiorilla (group 01011001) strikes for its clarity and ease of use. In particular, they walked the proverbial "extra mile" to include additional features like easy-to-use CLIs and Web GUIs.

After the examination, they agreed to further polish their project and publish it in [a github repository](https://github.com/methk/RaftShop) so that everyone interested in using Raft in their Jolie (and [Java](https://docs.jolie-lang.org/documentation/architectural_composition/embedding_java.html), and [JavaScript](https://docs.jolie-lang.org/documentation/architectural_composition/embedding_javascript.html)) projects can take it as a reference implementation. Technically, since in the project Raft is developed as a standalone set of microservices --- composed with the [embedding](https://docs.jolie-lang.org/documentation/architectural_composition/embedding.html) mechanism --- other programmers can easily extract and integrate them in their projects.

In addition, they also [wrote a thorough description](https://github.com/methk/RaftShop/wiki) of their project for an audience that has little to no knowledge of the Jolie language. In the document they both illustrate their design decisions and which features of the Jolie language they used in the implementation.

A genuine thank to Matteo, Arnaldo, and Salvatore which provided a significant contribution to the Jolie community. Keep up the good work!