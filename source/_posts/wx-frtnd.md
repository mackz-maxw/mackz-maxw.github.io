---
title: 微信小程序项目
lang: zh-CN
---

试着做一个微信小程序的项目，还不知道会遇到多少坑......

## 4/26

网页后端体系架构（自下而上）：

**数据库**：存储字段（列）组成的实体集（表）

通过MyBatisPlus将数据库实体集与Java对象建立联系，其中实体集中的字段与Java类中的属性一一对应。

**DAO层/Mapper层**：提供CRUD的Java方法，处理/提供data object

DO即数据库实体集对应的Java对象。

**Service层：**统一业务逻辑处理，为不同服务提供可复用的调用接口；为CRUD方法赋予实际业务意义；处理param数据，提供VO对象

将从controller层拿到的param数据转化成DO对象，传至DAO层；将DAO层提供的DO对象转化成VO对象，提供给controller层。

**Controller层：**分配URL，接受来自页面的请求，向下处理后，拿到返回数据返回页面。

从页面拿到param数据提供给service层，接受service层传递的VO对象给页面展示。



更多关于数据结构层的介绍：https://blog.csdn.net/qq_27022241/article/details/112002345