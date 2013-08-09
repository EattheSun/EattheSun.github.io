---
layout: post
title: GroupCache
---

## 和memcache的区别

* 整合了 client 和 server 端，在 memcache 中, server 和 client 端是相互分离的。整合起来后方便部署。

* 添加了Cache Filling 机制。在memcache中，没有命中cache，只会提示说：“Sorry, cache miss.”，解决了memcached中的"惊群效应"。



LRU cache


