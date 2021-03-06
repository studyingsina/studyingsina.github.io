---
layout: post
title: "MySQL Replace 自增问题"
categories: 
- Tech
tags:
- DB
---

* content
{:toc}

![递增](/css/pics/2018-02-01-mysql-replace-increament.jpg)

## 背景
今天线上环境发现一个用replace into 语句递增问题，我的理解是时间越早自增的id越我，但是这一例却是时间越早、自增的id反而大；

## 问题描述

业务上需要一个自增序号，于是就用MySQL InnoDB 表实现了一个；表结构如下：

```java

CREATE TABLE `seq` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `ref_id` bigint(20) NOT NULL COMMENT '引用id',
  `create_time` timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  PRIMARY KEY (`id`),
  UNIQUE KEY `index_ref_id` (`ref_id`)
) ENGINE=InnoDB AUTO_INCREMENT=5000 DEFAULT CHARSET=utf8mb4 COMMENT='唯一序号表';

```

然后程序中通过语句实现（业务上会存在同一refId重复提交，所以用的replace而不是insert）：

    replace into seq (ref_id) values (#{refId);

理论上时间越早、seq表的id就越小，但是今天发现两条记录刚好相反：（线上数据库的事务隔离级别为 REPEATABLE-READ，innodb_autoinc_lock_mode为1）

```java

mysql> select * from seq where id in (5237, 5238);
+------+--------+----------------------------+
| id   | ref_id | create_time                |
+------+--------+----------------------------+
| 5237 |   1433 | 2018-01-04 11:00:02.736458 |
| 5238 |   1396 | 2018-01-04 11:00:02.736385 |
+------+--------+----------------------------+

```

下面是两条数据插入时的Binlog日志：

```java

SET TIMESTAMP=1515034802.736385/*!*/;
BEGIN
/*!*/;
# at 959416466
#180104 11:00:02 server id 32145168  end_log_pos 959416533 CRC32 0xd98fea49     Rows_query
# replace into seq (ref_id)
#     values (1396)
# at 959416533
#180104 11:00:02 server id 32145168  end_log_pos 959416589 CRC32 0x3f19e645     Table_map: `top_ad_plan`.`seq` mapped to number 474
# at 959416589
#180104 11:00:02 server id 32145168  end_log_pos 959416648 CRC32 0xb50d056a     Write_rows: table id 474 flags: STMT_END_F
 
BINLOG '
sphNWh0Qf+oBQwAAANWILzmAACtyZXBsYWNlIGludG8gc2VxIChyZWZfaWQpCiAgICB2YWx1ZXMg
KDEzOTYpSeqP2Q==
sphNWhMQf+oBOAAAAA2JLzkAANoBAAAAAAEAC3RvcF9hZF9wbGFuAANzZXEAAwgIEQEGAEXmGT8=
sphNWh4Qf+oBOwAAAEiJLzkAANoBAAAAAAEAAgAD//h2FAAAAAAAAHQFAAAAAAAAWk2Ysgs8gWoF
DbU=
'/*!*/;
### INSERT INTO `top_ad_plan`.`seq`
### SET
###   @1=5238 /* LONGINT meta=0 nullable=0 is_null=0 */
###   @2=1396 /* LONGINT meta=0 nullable=0 is_null=0 */
###   @3=1515034802.736385 /* TIMESTAMP(6) meta=6 nullable=0 is_null=0 */
# at 959416648
#180104 11:00:02 server id 32145168  end_log_pos 959416679 CRC32 0x98567a1c     Xid = 414613759
COMMIT/*!*/;
# at 959416679
#180104 11:00:02 server id 32145168  end_log_pos 959416744 CRC32 0xc57084a5     GTID [commit=no]
SET @@SESSION.GTID_NEXT= 'f3aa695d-1a7e-11e7-84f7-00228ce195da:19314136'/*!*/;
# at 959416744
#180104 11:00:02 server id 32145168  end_log_pos 959416827 CRC32 0x67fab679     Query   thread_id=3064136       exec_time=0     error_code=0
SET TIMESTAMP=1515034802.736458/*!*/;
BEGIN
/*!*/;
# at 959416827
#180104 11:00:02 server id 32145168  end_log_pos 959416894 CRC32 0x120a343f     Rows_query
# replace into seq (ref_id)
#     values (1433)
# at 959416894
#180104 11:00:02 server id 32145168  end_log_pos 959416950 CRC32 0x22b0e030     Table_map: `top_ad_plan`.`seq` mapped to number 474
# at 959416950
#180104 11:00:02 server id 32145168  end_log_pos 959417009 CRC32 0x798a319a     Write_rows: table id 474 flags: STMT_END_F
  
BINLOG '
sphNWh0Qf+oBQwAAAD6KLzmAACtyZXBsYWNlIGludG8gc2VxIChyZWZfaWQpCiAgICB2YWx1ZXMg
KDE0MzMpPzQKEg==
sphNWhMQf+oBOAAAAHaKLzkAANoBAAAAAAEAC3RvcF9hZF9wbGFuAANzZXEAAwgIEQEGADDgsCI=
sphNWh4Qf+oBOwAAALGKLzkAANoBAAAAAAEAAgAD//h1FAAAAAAAAJkFAAAAAAAAWk2Ysgs8ypox
ink=
'/*!*/;
### INSERT INTO `top_ad_plan`.`seq`
### SET
###   @1=5237 /* LONGINT meta=0 nullable=0 is_null=0 */
###   @2=1433 /* LONGINT meta=0 nullable=0 is_null=0 */
###   @3=1515034802.736458 /* TIMESTAMP(6) meta=6 nullable=0 is_null=0 */
# at 959417009
#180104 11:00:02 server id 32145168  end_log_pos 959417040 CRC32 0xe8c81a16     Xid = 414613760
COMMIT/*!*/;
# at 959417040
#180104 11:00:02 server id 32145168  end_log_pos 959417105 CRC32 0x5b22e732     GTID [commit=no]
SET @@SESSION.GTID_NEXT= 'f3aa695d-1a7e-11e7-84f7-00228ce195da:19314137'/*!*/;
# at 959417105
#180104 11:00:02 server id 32145168  end_log_pos 959417187 CRC32 0xbd88c985     Query   thread_id=2955584       exec_time=0     error_code=0

```

## 原因

待查找；

## 结论
如果业务上需要实现自增，不要依赖数据库自增id去实现，因为不可靠，不一定时间越早的id就越小；

> You should never depend on the numeric features of autogenerated keys.

[《MySQL AUTO_INCREMENT does not ROLLBACK》](https://stackoverflow.com/questions/449346/mysql-auto-increment-does-not-rollback)
