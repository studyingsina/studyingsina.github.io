---
layout: post
title: "mysql db copy"
categories: 
- Tech
tags:
- DB
---

* content
{:toc}

### 背景
在测试环境有时候需要copy数据库，以前用过mysqldump，先将source数据导出来，再手工创建新的target数据库，再导入表结构及数据，在网上查了一下，发现mysqldbcopy命令可以将这事件搞定。

### 使用
mysqldbcopy --source=username:password@localhost --destination=username:password@localhost source_db:target_db

### 参考
https://dev.mysql.com/doc/mysql-utilities/1.5/en/utils-task-clone-db.html
