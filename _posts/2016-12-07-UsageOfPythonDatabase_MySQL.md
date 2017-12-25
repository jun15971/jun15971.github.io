---
layout: post
title: Python对于各数据库的用法
description: 调用MySQL数据库
date: 2016-12-07 14:43:15 +0800
image: assets/images/databaseicon.jpeg
---

python作为“胶水语言”具有很强的扩展性与其他多种编程语言及数据库都能很好的集成操作。本文主要讨论Python在数据库方面的操作。  
`Python`标准数据库接口为`Python DB-API`，该接口支持大多数主流数据库：`MySQL、Redis、Mangodb、sqlite、PostgreSQ、Oracle、Microsoft SQL`等等，具体可以访问[Python数据库借口及其API](https://wiki.python.org/moin/DatabaseInterfaces)查看具体列表。  


## Python操作MySQL

### 安装


### 异常处理  


|异常| 描述|
|---|---|
|warning| 当有严重警告时触发，例如插入数据是被截断等等。必须是 StandardError 的子类。|
|Error|警告以外所有其他错误类。必须是 StandardError 的子类。|
|InterfaceError|当有数据库接口模块本身的错误（而不是数据库的错误）发生时触发。 必须是Error的子类。|
|DatabaseError|和数据库有关的错误发生时触发。 必须是Error的子类。|
|DataError|当有数据处理时的错误发生时触发，例如：除零错误，数据超范围等等。 必须是DatabaseError的子类。|
|OperationalError	|指非用户控制的，而是操作数据库时发生的错误。例如：连接意外断开、 数据库名未找到、事务处理失败、内存分配错误等等操作数据库是发生的错误。 必须是DatabaseError的子类。|
|IntegrityError	|完整性相关的错误，例如外键检查失败等。必须是DatabaseError子类。|
|InternalError|数据库的内部错误，例如游标（cursor）失效了、事务同步失败等等。 必须是DatabaseError子类。|
|ProgrammingError|程序错误，例如数据表（table）没找到或已存在、SQL语句语法错误、 参数数量错误等等。必须是DatabaseError的子类。|
|NotSupportedError|不支持错误，指使用了数据库不支持的函数或API等。例如在连接对象上 使用.rollback()函数，然而数据库并不支持事务或者事务已关闭。 必须是DatabaseError的子类。|


## Python操作sqlite


## Python操作Redis


## Python操作Mangodb
