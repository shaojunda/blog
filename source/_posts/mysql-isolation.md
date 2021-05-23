---
title: MySQL Isolation
date: 2021-01-18 20:18:19
tags:
  - DB
  - MySQL
keywords:
  - MySQL
  - DB
  - Database
  - 隔离级别

description: MySQL 隔离性 隔离级别
---

今天复习了一下 MySQL 的隔离级别，SQL 标准的事务隔离级别包括：
- 读未提交（read uncommitted）：一个事务还没提交时，事务中所做的改动就能被其他事务看到。
- 读提交（read committed）：一个事务提交之后，事务中所做的改动才会被其他事务看到。
- 可重复读（repeatable read）：一个事务执行过程中所看到的数据与这个事务开始时看到的数据始终一致。未提交时该事务所做的改动其他事务也无法看到。
- 串行化（serializable）：对于用一行记录同一时刻只能有一个人操作，读操作会加读锁，写操作会加写锁，遇到锁冲突时，后访问的事务需要等前一个事务提交之后方可继续执行。

MySQL 和 PostgreSQL 的默认隔离级别都是「读提交（read committed）」，可以通过执行
`show variables like 'transaction_isolation';` 来查看。
