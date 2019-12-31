---
layout: post
title: Json 字符串在 spring cloud contracts yaml 中的
categories: [基础]
tags: [前端]
description: css 基础知识点
---

### 当直接使用body时，

当在 post 的 request 中，如想传递 json 字符串，可以使用

```yml
body：

  example: ‘{\"test\":\"1\"}’

```



当 json 字符串在 response 中，同样的方法就不适用了，生成的结果包含两次转义。

目前该问题暂时无解，后续进展可关注该问题 issue.



一个解决方式就是使用 bodyFromFile.

