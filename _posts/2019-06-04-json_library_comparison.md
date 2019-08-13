---
layout: post
title: json-simple vs Gson vs jackson vs fastjson
categories: [Tech]
tags: [tool]
description: 对几种主流的 json 处理库进行的比较
---

本文对四种比较主流的 json 解析和反解析的库做了简单的比较。这四个json 库分别为：

[google Gson](<https://github.com/google/gson>)

[FastXML jackson](<https://github.com/FasterXML/jackson-databind>)

[Alibaba fastjson](<https://github.com/alibaba/fastjson>)

[json simple](<https://github.com/fangyidong/json-simple>)

### 流行程度

以下为 2019年6月5日这四个库的 github 统计数据截图。

![](https://git-page.oss-cn-chengdu.aliyuncs.com/json-compare/fastjson.png)

![](https://git-page.oss-cn-chengdu.aliyuncs.com/json-compare/gson.png)

![](https://git-page.oss-cn-chengdu.aliyuncs.com/json-compare/json-simple.png)

![](https://git-page.oss-cn-chengdu.aliyuncs.com/json-compare/jackson.png)

从使用数量来看， jackson 是被使用最多的库。而关注率最高的是 alibaba  的 fastjson。紧跟着是 google 的 Gson。

### 序列化和反序列化

以下测试结果基于 jackson: 2.9.9, gson: 2.8.5, fastjson: 1.2.58, son-simple: 2.8.5。

#### 序列化

json-simple 只能处理 JSONArray 或 JSONObject 对象，不能直接对类对象进行序列化和反序列化操作。故一下只比较 Jackson, Gson, fastJson。

Jackson, Gson, fastJson 序列化类对象的代码都很简单，基本两行就能搞定。

Jackson

```java
  ObjectMapper mapper = new ObjectMapper(); 
  System.out.println(mapper.writeValueAsString(book));
```

Gson

```
  Gson gson = new Gson();
  gson.toJson(book);
```

fastJson

```
 JSON.toJSONString(book)
```

以下代码使用 Jackson, Gson 和 fastJson 的默认配置序列化了同一个对象。

![](https://git-page.oss-cn-chengdu.aliyuncs.com/json-compare/serialize_object.png)

其输出如下：

![](https://git-page.oss-cn-chengdu.aliyuncs.com/json-compare/object_serialize_ouput.png)

由上输出结果可见，其中对于为 null 的属性， boolean 类型以 is 开头的属性以及输出的内容，各个库是略有不同的。

|                                  | jackson       | Gson       | fastJson      |
| -------------------------------- | ------------- | ---------- | ------------- |
| 对 null field 的处理             | 保留          | 忽略       | 忽略          |
| boolean 字段以is开头时输出字段名 | 去掉 is 前缀  | 原字段名   | 去掉 is 前缀  |
| 序列化时输出字段的规则           | 根据 get 方法 | 根据 field | 根据 get 方法 |

##### 序列化日期类型

以下代码对各种时间类型进行了序列化输出：

![](https://git-page.oss-cn-chengdu.aliyuncs.com/json-compare/serialize_date.png)

其输出如下：

```json
=========jackson json result========
{"date":61514956800000,"time":"03:04:03","localDate":{"year":2019,"month":"APRIL","monthValue":4,"dayOfMonth":3,"dayOfWeek":"WEDNESDAY","era":"CE","dayOfYear":93,"leapYear":false,"chronology":{"id":"ISO","calendarType":"iso8601"}},"localDateTime":{"nano":3,"year":2019,"monthValue":4,"dayOfMonth":3,"hour":8,"minute":40,"second":35,"month":"APRIL","dayOfWeek":"WEDNESDAY","dayOfYear":93,"chronology":{"id":"ISO","calendarType":"iso8601"}},"localTime":{"hour":12,"minute":40,"second":3,"nano":34}}
=====Gson result=======
{"date":"May 3, 3919, 12:00:00 AM","time":"03:04:03 AM","localDate":{"year":2019,"month":4,"day":3},"localDateTime":{"date":{"year":2019,"month":4,"day":3},"time":{"hour":8,"minute":40,"second":35,"nano":3}},"localTime":{"hour":12,"minute":40,"second":3,"nano":34}}
=====fast json result=======
{"date":61514956800000,"localDate":"2019-04-03","localDateTime":"2019-04-03T08:40:35.000000003","localTime":"12:40:03.000000034","time":7239843000}

```

序列化时间类型时，除了 Date 类型外，Jackson 和 Gson 都把其余时间类型当成对象输出。

|               | jackson               | Gson                                                    | fastJson                                                     |
| ------------- | --------------------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| Date          | 时间戳, 为 getTime 值 | 默认使用 US format  输出日期，格式 MMM d, yyy h:mm:ss a | 时间戳，为 getTime 值                                        |
| LocalDate     | 对象                  | 对象                                                    | yyyy-MM-dd 格式                                              |
| LocalDateTime | 对象                  | 对象                                                    | 完整格式为 yyyy-MM-dd'T'hh:mm:ss.SSSSSSSSS<br />输出根据输入最低位而变化，如构造为LocalDateTime.of(2019, 4, 3, 8, 40, 35, 3)， 则输出为 2019-04-03T08:40:35.000000003。<br />如构造为LocalDateTime.of(2019, 4, 3, 8, 40)， 则输出为 2019-04-03T08:40。 |
| LocalTime     | 对象                  | 对象                                                    | 完整格式为 hh:mm:ss.SSSSSSSSS ，输出同样以构造时的最低位结尾。 |

#### 反序列化

同样  json simple 不支持直接将 string 转为指定对象，只能转为 JsonObject 或 JsonArray。json simple 反序列化代码如下。对于数字类型，反序列话后默认是 long 类型。

```
JSONParser jsonParser = new JSONParser();
JSONObject stu = (JSONObject) jsonParser.parse(jsonStr);  // 如果为list则转为 JSONArray
```

反序列化时各库的默认行为：

|                         | Jackson                                       | Gson   | fast Json |
| ----------------------- | --------------------------------------------- | ------ | --------- |
| string中包含多余的field | 抛 UnrecognizedPropertyException              | 忽略   | 忽略      |
| 接收对象的 setter 方法  | 不要求setter，当有 setter 时，优先使用 setter | 不要求 | 要求      |

##### 反序列化布尔值

对于 boolean 值进行处理时，如果属性名称以 is 开头，各库的处理情况会略有不同。

下面以一个 hot 字段为例，比对各库在反序列化时对 boolean 的处理，第一列表示反序列时的情况，第一行表示使用的 json 库，交叉格表示在给的反序列情况下，使用给定的 json 库反序列化后对象中字段的值或抛出的异常。

|                                     | Jackson                       | Gson | fastJson |
| ----------------------------------- | ----------------------------- | ---- | -------- |
| String 中 isHot  -> 对象 hot 字段   | UnrecognizedPropertyException | null | true     |
| String 中 hot  -> 对象 isHot 字段   | true                          | null | null     |
| String 中 isHot  -> 对象 isHot 字段 | true                          | true | true     |
| String 中 hot  -> 对象 hot 字段     | true                          | true | true     |

由于序列化时，jackson 和 fastjson 都会将以 is 开头的 field 的前缀 is 去掉。基于这一前提，Jackson 仍能正常工作，但是 fastjson 不能正常反序列化对象，其值会为 null。Gson 序列化时是保持的 field 原名称，故始终可以正常工作。 

##### 反序列化日期时间类型

使用如下代码测试了序列化后的日期是否能成功反序列化。

![](https://git-page.oss-cn-chengdu.aliyuncs.com/json-compare/deserialize_date.png)

其中，Gson 和 fastJson 都可以成功反序列化自己序列化的各种日期类型。Jackson 在默认情况下，能成功反序列化自己序列化的 Date 和 Time 类型，但不能反序列化 LocalDate, LocalDateTime 和 LocalTime 等类。

#### 修改默认配置

以上的比较都是基于 Jackson, Gson 和 fastJson 的默认配置，各个库也支持更改配置。以下以一些简单的示例代码展示如何更改默认配置。

Jackson: 直接配置 ObjectMapper。以下例子忽略值为 null 的字段。

```java
ObjectMapper mapper = new ObjectMapper();
mapper.setSerializationInclusion(Include.NON_NULL);
mapper.writeValueAsString(book);
```

Gson ：在创建 Gson 对象时配置。以下例子允许输出值为 null 的字段。

```java
Gson gson = new GsonBuilder().serializeNulls().create();
gson.toJson(book);
```

fastJson: 在序列化时添加 SerializerFeature， 目前一共提供 30 种 Feature。以下例子允许输出值为 null 的字段。

```java
JSON.toJSONString(dateCollection, SerializerFeature.WriteMapNullValue)
```

以上测试的完整代码请参见[github](https://github.com/LindseyZhang/json-lib-compare)

#### ObjectMapper 和 Gson 能否做成全局的

每次使用都构建 ObjectMapper 和 Gson 对象，显然成本太高了。那么这两个对象能否做成全局的呢。

根据 Jackson 的官方文档：

> Mapper instances are fully thread-safe provided that ALL configuration of the instance occurs before ANY read or write calls. If configuration of a mapper instance is modified after first usage, changes may or may not take effect, and configuration calls themselves may fail. If you need to use different configuration, you have two main possibilities:

ObjectMapper 是线程安全的。配置只有在第一次 read 或 write 前设置的才生效，使用过 read 或 write 后修改配置可能不生效。如果你全局都使用的是同一配置的 ObjectMapper, 那么是非常推荐将该变量做成全局并共享的。

根据 Gson 官方文档：

> Gson instances are Thread-safe so you can reuse them freely across multiple threads.

Gson 也是线程安全的，使用时如果都是同一配置，也建议做成全局共享的，避免每次使用时构造的开销。

### 处理速度比较

根据线上网友做的比较，其处理速度大概如下：

(~> 表示略大于)`

处理大文件： Jackson ~> Json-simple > Gson > JsonP  

处理小文件：Gson > Json-simple ~> Jason > Jackson

建议：如果系统处理的 json 都比较大，选择 Jackson；如果处理的 json都比较小，选择Gson; 如果处理的 json 大小不定，则选择 Json-simple 以达到最优的平均性能。





reference:

[fastJson 内幕](https://wenshao.iteye.com/blog/1142031/)

[JSON library 性能比较](https://blog.overops.com/the-ultimate-json-library-json-simple-vs-gson-vs-jackson-vs-json/)



