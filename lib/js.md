# 从csv到CouchDB的过程

## 目录

* [流程](#流程)
* [JSON](#json)
* [CouchDB](#couchdb)
  * [问题处理流程](#问题处理流程)
* [服务器](#服务器)

## 流程

[策划所配置的数据](http://192.168.1.170:8081/release/config)是以csv为格式的文本表，我们目前通过如下的方式将配置表转换为代码中可以使用的数据。

```text
csv --> JSON --> CouchDB --> 客户端、服务器
```

## JSON

我们通过JSON将配置表转换为大部分NoSQL皆可直接读取的内容，JSON文件存在于<http://192.168.1.170:8081/tools/config-mapping> ，可用Git拉取。以下为掌沃无限JSON编码规范,编码时注意分支。

```json
//s世界等级.json
[
  { //两格缩进
    "jsonFileName": "world_level.json", //采用单词全小写加下划线(unix_like)命名
    "docName": "world_level:id",    //名字与上面一致
    "shouldObject": false,
    "sheet": "s世界等级",   //与csv配置表的名字对应，否则无法转表
    "mapping": {
      "id": "天数", //前面的名字自取，采用单词全小写加下划线(unix_like)命名
      "world_level": "世界等级" //后面的与csv配置表中的属性(每一列的名字)一一对应，尽量保证顺序对应
    },
    "type": {
      "id": "int",  //与mapping的命名一致
      "world_level": "int"  //类型可用int, float, string
    }
  }
]   //mapping中的id会成为CouchDB中的key值，所以尽量选择csv中的索引(或者易于区分且唯一的值)作为JSON的id
```

完成编码后，需要push才可在CouchDB中显现效果。

## CouchDB

CouchDB中通过JavaScript语句获取到JSON转表的数据，每一个JSON对应了两个JS文件，分别用于客户端和服务器。其中，客户端的JS文件名以c_起头，而服务器的JS文件则直接与JSON对应。例如：(客户端：c_world_level， 服务器：world_level)。以下为掌沃无限JavaScript编码规范,编码时注意分支。

```javascript
function (doc) {
    var [type, id] = doc._id.split(':')
    if (type === 'world_level') {   //与JSON的docName对应
      var config = doc.value
      //客户端需要将键的首字母大写，而服务器不需要
      var {uppercaseFirst} = require('views/lib/common')
      //其他常用函数，可在CouchDB的离散数据表中查询
      //var {parse2DArray} = require('views/lib/common')
      //var {currencies} = require('views/lib/common')
      //var {findPropertyGroup} = require('views/lib/common')
      var {
        world_level //与JSON的mapping一致
      } = config
  
      var output = {    //CouchDB所显示的内容
        id: parseInt(id),
        world_level: world_level  //output里的键应与JSON一致
      }
      var out = uppercaseFirst(output)  //将键的首字母大写
  
      emit(parseInt(id), out)
    }
  }
```

### 问题处理流程

1. 如果将JS代码保存后，仍然无法在CouchDB中看到配置表数据时，先检查JS代码(主要是type、output)有没有bug
2. 如果JS代码没有问题，但仍然无法获取到配置表的数据时，可能是配置表更新后，没有在[配置表那个网站](http://192.168.1.102:8081/)刷新数据
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
