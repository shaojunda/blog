---
title: Cache Design Patterns
date: 2020-12-20 23:50:50
tags:
  - cache
  - programing
keywords:
  - cache
  - Cache Aside
  - Read Through
  - Write Through
  - Write back

description: cache design patterns
---

一个项目中之前缓存的设计有一些缺陷，所以重新学习了一下缓存设计的一些常见套路，准备重新设计一下项目中的缓存方案。

## Cache Aside
一种比较常见的缓存模式。
1. 数据读取策略
  * 应用先向缓存服务请求数据；
  * 如果有，直接返回数据给请求方；
  * 如果没有，则去数据库中查询结果并返回给请求方；
  * 应用更新数据到缓存中。

2. 数据更新策略
  * 应用先更新数据库；
  * 应用再令缓存失效。

![](/images/cache-aside.png)
<!--more-->

**注意** 这里一定要先更新数据库再令缓存失效，不然可能出现数据不一致的情况。比如进程 A 更新数据，先令缓存失效了这时进程 B 来读缓存发现没有去数据库中拿数据，此时数据库还没有更新这是 B 进程拿到的就是旧数据并且写入到缓存中。

## Read Through
用来处理读取数据的场景，应用只能看到缓存服务，数据库对应用来说是透明的。
### 数据读取策略
  * 应用向缓存服务请求数据；
  * 如果有，直接返回数据给请求方；
  * 如果没有，缓存服务查询数据库，并更新缓存；
  * 缓存服务将数据返回请求方。

## Write Through
用来处理数据更新的场景。
### 数据更新策略
  * 应用向缓存服务发出写请求；
  * 如果有，缓存服务先更新缓存中数据，再更新数据库；
  * 如果没有，缓存服务直接更新数据库；
  * 缓存服务告知应用更新完成。

![](/images/read-write-through.png)

## Write Back
在更新数据的时候，只更新缓存，不更新数据库。缓存服务异步批量地更新数据库。这样就可以拥有较大的请求吞吐能力，也不用担心数据库短时间无法访问，但是缺点也很明显，数据不是强一致的且容易丢失。


