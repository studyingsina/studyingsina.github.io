---
layout: post
title: "MySQL JDBC 更新数据丢失毫秒精度"
description: "Java Mybatis JDBC 更新数据丢失毫秒精度"
categories: 
- Tech
tags:
- Tech
---

* content
{:toc}

![MySQL-Lose-Precision](/css/pics/2017-12-25-mysql-lose-precision.jpg)

## 背景

业务上需要将MySQL表中的时间精度由原来的“年月日 时分秒”精确到毫秒，但是发现通过MySQL客户端update语句可以更新毫秒成功，但是通过程序却不行，毫秒数始终为0；

## 原因

经过以上背景描述，第一反应是用程序更新数据库有问题，将精度给搞丢了；我们数据库组件这块用的：Mybatis+C3p0+MySQL Connector，首先将更新语句日志打出来：

```
2017-12-22 17:27:21,719-[TS] DEBUG main updateByPrimaryKeySelective - ==>  Preparing: update schedule SET version = version + 1, apply_time = ? where id = ?
2017-12-22 17:27:21,732-[TS] DEBUG main updateByPrimaryKeySelective - ==> Parameters: 2017-12-22 17:27:19.579(Timestamp), 1049(Long)
2017-12-22 17:27:30,033-[TS] DEBUG main updateByPrimaryKeySelective - <==    Updates: 1

```

发现在Mybatis这一层，精度是传递下去了；

所以去Debug下，看看到底在哪一行把这精度给搞丢了，当我们一步步跟到MySQL JDBC Driver这一个方法时，发现：

```Java

private synchronized void setTimestampInternal(int parameterIndex, Timestamp x, Calendar targetCalendar, TimeZone tz, boolean rollForward) throws SQLException {
    if(x == null) {
        this.setNull(parameterIndex, 93);
    } else {
        this.checkClosed();
        if(!this.useLegacyDatetimeCode) {
            this.newSetTimestampInternal(parameterIndex, x, targetCalendar);
        } else {
            String timestampString = null;
            Calendar sessionCalendar = this.connection.getUseJDBCCompliantTimezoneShift()?this.connection.getUtcCalendar():this.getCalendarInstanceForSessionOrNew();
            synchronized(sessionCalendar) {
                x = TimeUtil.changeTimezone(this.connection, sessionCalendar, targetCalendar, x, tz, this.connection.getServerTimezoneTZ(), rollForward);
            }

            if(this.connection.getUseSSPSCompatibleTimezoneShift()) {
                this.doSSPSCompatibleTimezoneShift(parameterIndex, x, sessionCalendar);
            } else {
                synchronized(this) {
                    if(this.tsdf == null) {
                        this.tsdf = new SimpleDateFormat("\'\'yyyy-MM-dd HH:mm:ss\'\'", Locale.US);
                    }

                    timestampString = this.tsdf.format(x);
                    this.setInternal(parameterIndex, timestampString);
                }
            }
        }

        this.parameterTypes[parameterIndex - 1 + this.getParameterIndexOffset()] = 93;
    }

}

```

此处有一个格式化代码，将时间对象都以'yyyy-MM-dd HH:mm:ss'形式给处理了，自然将我们的毫秒数给抛弃了，此时查看我们的JDBC Driver版本，是5.1.19，难道数据库支持毫秒而驱动不支持？不太可能，所以怀疑是我们的驱动版本过低，因为支持毫秒的特性是[MySQL 5.6.4](https://dev.mysql.com/doc/refman/5.6/en/fractional-seconds.html)开始有的，所以将JDBC Driver升级一下，这里我用的5.1.39，再跑一遍程序，毫秒数已经可以插入到数据库，再次Debug下，可以看到在相同的方法中，MySQL已经加上了毫秒数的支持：

```
if (this.connection.getUseSSPSCompatibleTimezoneShift()) {
    doSSPSCompatibleTimezoneShift(parameterIndex, x);
} else {
    synchronized (this) {
        if (this.tsdf == null) {
            this.tsdf = new SimpleDateFormat("''yyyy-MM-dd HH:mm:ss", Locale.US);
        }

        StringBuffer buf = new StringBuffer();
        buf.append(this.tsdf.format(x));

        if (this.serverSupportsFracSecs) {
            int nanos = x.getNanos();

            if (nanos != 0) {
                buf.append('.');
                buf.append(TimeUtil.formatNanos(nanos, this.serverSupportsFracSecs, true));
            }
        }

        buf.append('\'');

        setInternal(parameterIndex, buf.toString()); // SimpleDateFormat is not
        // thread-safe
    }
}

```

其实就是判断一下数据版本，如果支持毫秒精度特性，就把毫秒数加上；

## 解决

上边排查到原因，解决方法就好了，直接升级更新的驱动版本；

## 思考

其实这种问题在日常开发中很常见，保持一颗刨根问底的心很重要，太阳底下无新事、源码之下无黑盒，一步步追踪下去，会找到问题所在的。
