# 掌沃无限编码规范

## JSON

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

## JavaScript

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
        world_level: world_level    //output里的键应与JSON一致
      }
      var out = uppercaseFirst(output)  //将键的首字母大写
  
      emit(parseInt(id), out)
    }
  }
```

## Elixir

---
