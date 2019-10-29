# 从csv到CouchDB的过程

## 目录

* [流程](#流程)
* [JSON](#json)
* [CouchDB](#couchdb)
  * [问题处理流程](#问题处理流程)
* [服务器](#服务器)

## 流程

[策划所配置的数据](http://192.168.1.170:8081/my2/csv.git)是以csv为格式的文本表，我们目前通过如下的方式将配置表转换为代码中可以使用的数据。

```text
csv --> JSON --> CouchDB(JS) --> 客户端、服务器
```

## JSON

我们通过JSON将配置表转换为大部分NoSQL皆可直接读取的内容，JSON文件存在于<http://192.168.1.170:8081/my2/config-mapping> ，可用Git拉取。</br>
关于掌沃无限JSON编码规范请参见文档[JSON编码规范](/lib/code_guide.md#json)。</br>
完成编码后，需要push才可在CouchDB中显现效果。

## CouchDB

CouchDB中通过JavaScript语句获取到JSON转表的数据，每一个JSON对应了两个JS文件，分别用于客户端和服务器。其中，客户端的JS文件名以c_起头，而服务器的JS文件则直接与JSON对应。例如：(客户端：c_world_level， 服务器：world_level)。</br>
关于掌沃无限JSON编码规范请参见文档[JavaScript编码规范](/lib/code_guide.md#javascript)。

### 问题处理流程

1. 如果将JS代码保存后，仍然无法在CouchDB中看到配置表数据时，先检查JS代码(主要是type、output)有没有bug
2. 如果JS代码没有问题，但仍然无法获取到配置表的数据时，可能是配置表更新后，没有在[配置表那个网站](http://192.168.1.238:8081/)刷新数据
3. 如果刷新后仍然没有数据，则检查分支是否不一致
4. 如果分支也是一致的，那么检查JSON代码是否有bug

## 服务器

在服务器代码中，我们通过以下方式从CouchDB中获取游戏所需的数据：

```elixir
GameDef.defconf view: "deamon/slots", getter: :config
GameDef.defconf view: "deamon/star", getter: :star_config
GameDef.defconst path: "deamon/config/discrete_data", getter: :dis_config
```

---
