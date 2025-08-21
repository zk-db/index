---
title: RESTful
date: 2022-06-01 09:11:01
tags:
  - RESTful
  - HATEOAS
---

# REST

## `Resource Representational State Transfer`：

- **资源（Resource）** ：URL即资源，指向具体操作的对象
- **表现形式（Representational）**：请求数据类型（`json`, `xml`,`file`等）
- **状态转移（State Transfer）** ：通过具体的`method`行为（`GET`,`POST`,`PUT`,`PATCH`,`DELETE`）操作资源并改变资源状态

<!--more-->

## 动作

- `GET` 获取资源
- `POST` 保存资源
- `PUT` 更新资源（全量更新）
- `PATCH` 更新资源（部分更新）
- `DELETE` 删除资源

## URL设计规则

- 不可包含动词，仅描述资源
- 使用小写字母
- 设置版本号规则，实现版本化

## 

# HATEOAS

`Hypermedia as the Engine of Application State`

可以被简单的理解为为 REST API 中的 Resource 提供必要的链接，对，就像是 HTML 页面上的链接。我们在访问一个 web 站点的时候从来没有说要看一个说明文档并在其中找到我们所需要的资源的 URI，而是通过一个入口页面（当然，搜索引擎也提供了入口）所包含的链接，一步一步找到我们想要的内容。HATEOAS 是 REST 架构风格重要的组成部分，然而对于现在的诸多 REST 接口中却并没有它的身影。它被 [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html) 定义为 REST 的最终形态。[参考链接](https://blog.aisensiy.me/2017/06/04/spring-boot-and-hateoas/)



# SpringBoot

参考1：[SpringBoot Restful](https://spring.io/guides/tutorials/rest/)

参考2：[SpringBoot & HATEOAS](https://blog.aisensiy.me/2017/06/04/spring-boot-and-hateoas/)

参考3：[Spring HATEOAS](https://docs.spring.io/spring-hateoas/docs/1.5.0/reference/html/)

